---
title: "[iOS] Out Of Index Crash In UITableView"
excerpt: "Out Of Index Crash In UITableView"
description: "Out Of Index Crash In UITableView"
modified: 2024-02-22
categories: "iOS"
tags: [iOS, UITableView, OutOfIndex]

toc: true

header:
  teaser: /assets/images/teaser/ios-teaser.png
---

# 이슈
- UITableView의 DataSource에서 IndexPath를 사용하면서 Out Of Index 크래시 오류가 발생하는 상황
- UITableView 가 Thread 상에서 어떻게 동작 하는지 살펴볼 필요 있음

# 예시코드
```swift
class MainViewController {
    let tableView: UITableView
    let dataList: [String] = []

    ...

    func fetch() {
        getDataResource { newDataList in
            self.dataList = newDataList

            DispatchQueue.main.async {
                self.tableView.reloadData()
            }
        }
    }
}

extension MainViewController: UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return dataList.count
    }

    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = UITableViewCell()

        let data = dataList[indexPath.row] // crash 발생 가능성 있음
        cell.textLabel?.text = data

        return cell
    }
}

```

# 크래시 발생 케이스
- 코드상으로는 numberOfRowsInSection에서 사용한 dataList.count 갯수만큼, cellForRowAt에서 dataList에 접근하므로 크래시가 발생할 이유가 없어 보임
- 하지만, UITableView가 Main Thread 안에서 Reload되는 동작을 고려해 본다면, 특정 케이스에서 크래시가 발생할 수 있음
- 위 코드상에서 `fetch안의 getDataResource` 가 async하게 여러번 호출된다고 가정해보면(API Network 호출등), `tableView.reloadData()`은 async하게 여러번 호출되겠지만, UI업데이트는 Main Thread에서 동작하기때문에, Serial하게 한번씩 reload가 실행이 됨. 이때 `fetch()`안에서 `dataList` 의 갯수가 변경되게 되면, 이에 해당하는 `tableView.reloadData()`가 아직 Main Thread상에서 실행되지 않은 상태에서, `cellForRowAt` 에서 변경된 `dataList`를 사용할 수 있는 케이스가 발생. 이럴경우 cell의 갯수는 변경전의 dataList 갯수를 사용하고, 직접 cell을 그리는 와중에서는 변경된 dataList로 접근하면서, 원래 cell count 갯수와는 다른 dataList에 접근하면서 `Out of Index` 크래시가 발생할 수 있다.

# 방어로직
- 이를 방지하기위해, Array를 사용할때 Range범위를 체크하는 방어로직을 추가해야 함

```swift
extension MainViewController: UITableViewDataSource {
    ...

    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = UITableViewCell()

        guard dataList.indices.contains(indexPath.row) else { // dataList Array의 Range안에서 indexPath.row값을 사용할 수 있는지 체크하는 방어로직 필요
            return cell
        }

        let data = dataList[indexPath.row]
        cell.textLabel?.text = data

        return cell
    }
}

```

# Referrence
- [https://velog.io/@wansook0316/Out-Of-Index-In-Main-Async](https://velog.io/@wansook0316/Out-Of-Index-In-Main-Async){: target="_blank"}
