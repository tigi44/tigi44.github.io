---
title: "[WWDC21] Meet async/await in Swift"
excerpt: "[WWDC21] Meet async/await in Swift"
description: "[WWDC21] Meet async/await in Swift"
modified: 2021-07-02
categories: "WWDC21"
tags: [WWDC21, Swift, Swift 5.5, async/await]

header:
  teaser: /assets/images/teaser/wwdc21-teaser.jpg
---

# Functions
## synchronous
- blocks threads
![synchronous](/assets/images/post/wwdc21/asyncawait/synchronous.png)

## asynchronous
- use a completion handler
![asynchronous](/assets/images/post/wwdc21/asyncawait/asynchronous.png)

## Fetching a thumbnail
![fetching](/assets/images/post/wwdc21/asyncawait/fetching.png)
![fetchingcode](/assets/images/post/wwdc21/asyncawait/fetchingcode.png)

- sync threads : `thumbnailURLRequest`, `UIImage(data:)`
- async threads : `dataTask(with:completion:)`, `prepareThumnail(of:completionHandler:)`

# Async/Await

- async : enable a function to suspend
- await : marks where an async function may suspend execution
- Once an awaited async call completes, execution resumes after the await

```swift
func fetchThumbnail(for id: String) async throws -> UIImage {
  let request = thumbnailURLRequest(for: id)
  let (data, response) = try await URLSession.shared.data(for: request)
  guard (reponse as? HTTPURLResponse)?.statusCode == 200 else { throw FetchError.badID }
  let maybeImage = UIImage(data: data)
  guard let thumbnail = await maybeImage?.thumbnail else { throw FetchError.badImage }
  return thumbnail
}
```

- threads : `fetchThumbnail` > `thumbnailURLRequest(for:)` > `fetchThumbnail` > `data(for:)` > ... > `fetchThumbnail` > `UIImage(data:)` > `fetchThumbnail` > `thumbnail` > ... > `fetchThumbnail`
- awaitable(unblock thread ) : `thumbnailURLRequest(for:)`, `thumbnail`
- make it safer, shorter, better reflect your intent

```swift
extension UIImage {
  var thumbnail: UIImage? {
    get async {
      let size = CGSize(width: 40, height: 40)
      return await self.byPreparingThumbnail(ofSize: size)
    }
  }
}
```

## Async sequence

- `await` works in `for`loops
```swift
for await id in staticImageIDsURL.lines {
  let thumbnail = await fetchThumbnail(for: id)
  collage.add(thumbnail)
}
let result = await collage.draw()
```

## Normal function call

![normalfunction](/assets/images/post/wwdc21/asyncawait/normalfunction.png)

## An asynchronous function call

![asynchronousfunction](/assets/images/post/wwdc21/asyncawait/asynchronousfunction.png)

# Testing async code
- async makes testing a snap

- XCTestExpectation
```swift
class MockVuewModelSpec: XCTestCase {
  func testFetchThumbnails() throws {
    let expectation = XCTestExpectation(description: "mock thumbnails completion")
    self.mockViewModel.fetchThumbnail(for: mockID) { result, error in
      XCTAssertNil(error)
      expection.fulfill()
    }
    wait(for: [expection], timeout: 5.0)
  }
}
```

- async
```swift
class MockVuewModelSpec: XCTestCase {
  func testFetchThumbnails() async throws {

    XCTAssertNoThrow(try await self.mockViewModel.fetchThumbnail(for: mockID))

  }
}
```

# Bridging from sync to async

- Concurrency SwiftUI

- sync
```swift
struct ThumbnailView: View {
  @ObservedObject var viewModel: ViewModel
  var post: Post
  @State private var image: UIImage?

  var body: some View {
    Image(uiImage: self.image ?? placeholder)
      .onAppear {
        self.viewModel.fecthThumbnail(for: post.id) { result, _ in
          self.image = result
        }
      }
  }
}
```

- to async
```swift
struct ThumbnailView: View {
  @ObservedObject var viewModel: ViewModel
  var post: Post
  @State private var image: UIImage?

  var body: some View {
    Image(uiImage: self.image ?? placeholder)
      .onAppear {
        async {
            self.image = try? await self.viewModel.fecthThumbnail(for: post.id)
        }
      }
  }
}
```

# Async alternatives and continuations


## Continuations

- Continuations must be resumes exactly once on every path

![continuations](/assets/images/post/wwdc21/asyncawait/continuations.png)

- function old code
```swift
func getPersistentPosts(completion: @escaping ([Post], Error?) -> Void) {
  do {
    let req = Post.fetchRequest()
    req.sortDescriptor = [NSSortDescriptor(key: "date", ascending: true)]
    let asyncRequest = NSAsynchronousFetchRequest<Post>(fetchRequest: req) { result in
      completion(result.finalResult ?? [], nil)
    }
    try self.managedObjectContext.execute(asyncRequest)
  } catch {
    completion([], error)
  }
}
```
- continuations code
```swift
func persistentPosts() async throws -> [Post] {
  typealias PostContinuation = CheckedContinuation<[Post], Error>
  return try await withCheckedThrowingContinuation { (continuation: PostContinuation) in
    self.getPersistentPosts { posts, error in
      if let error = error {
        continuation.resume(throwing: error)
      } else {
        continuation.resume(returning: posts)
      }
    }
  }
}
```

## Continuations in Delegate

```swift
class ViewController: UIViewController {
  private var activeContinuation: CheckedContinuation<[Post], Error>?
  func sharedPostsFromPeer() async throws -> [Post] {
    try await withCheckedThrowingContinuation { continuation in
      self.activeContinuation = continuation
      self.peerManager.syncSharedPosts()
    }
  }
}

extension ViewController: PeerSyncDelegate {
  func peerManager(_ manager: PeerManager, received posts: [Post]) {
    self.activeContinuation?.resume(returing: posts)
    self.activeContinuation = nil // guard against multiple calls to resume
  }

  func peerManager(_ manager: peerManager, hadError error: Error) {
    self.activeContinuation?.resume(throwing: error)
    self.activeContinuation = nil
  }
}
```

# Reference
- [https://developer.apple.com/wwdc21/10132](https://developer.apple.com/wwdc21/10132){:target="_blank"}
