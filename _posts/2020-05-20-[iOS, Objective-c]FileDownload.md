---
layout: post
categories: "iOS"
title: "[iOS, Objective-c] File Download (HTTP Response attachment)"
description: "File Download (HTTP Response attachment)"
modified: 2020-05-20
tags: [iOS, Objective-c, File Download, HTTP Response attachment]
---

## File Download
```obj-c

+ (void)downloadTaskFromRequest:(NSURLRequest *)aRequest fileName:(NSString *)aFileName
{
    NSURLSessionDownloadTask *sDownloadTask = [[NSURLSession sharedSession] downloadTaskWithRequest:aRequest
                                                                                  completionHandler:^(NSURL *aLocation, __unused NSURLResponse *aResponse, NSError *aError) {

        if (!aError && aLocation)
        {
            BOOL    sIsSuccess          = NO;
            NSError *sRemoveFileError   = nil;
            NSError *sMoveFileError     = nil;

            NSFileManager   *sFileManager           = [NSFileManager defaultManager];
            NSArray         *sPaths                 = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
            NSString        *sDocumentsDirectory    = [sPaths objectAtIndex:0];
            NSString        *sFileName              = aFileName.length > 0 ? aFileName : aResponse.suggestedFilename;
            NSString        *sFilePath              = [sDocumentsDirectory stringByAppendingPathComponent:sFileName];

            if ([sFileManager fileExistsAtPath:sFilePath])
            {
                [sFileManager removeItemAtPath:sFilePath error:&sRemoveFileError];
            }

            sIsSuccess = [sFileManager copyItemAtPath:aLocation.path toPath:sFilePath error:&sMoveFileError];
        }
    }];

    [sDownloadTask resume];
}

```
```obj-c

+ (void)dataTaskFromRequest:(NSURLRequest *)aRequest fileName:(NSString *)aFileName
{
    NSURLSessionConfiguration *sSessionConfiguration = [NSURLSessionConfiguration defaultSessionConfiguration];
    NSURLSession *sSession = [NSURLSession sessionWithConfiguration:sSessionConfiguration];

    NSURLSessionDataTask *sPostDataTask = [sSession dataTaskWithRequest:aRequest
                                                    completionHandler:^(NSData *aData, NSURLResponse *aResponse, NSError *aError) {

            if (!aError) {

                NSArray  *sPaths                = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
                NSString *sDocumentsDirectory   = [sPaths objectAtIndex:0];
                NSString *sFileName             = aFileName.length > 0 ? aFileName : aResponse.suggestedFilename;
                NSString *sFilePath             = [sDocumentsDirectory stringByAppendingPathComponent:sFileName];

                [aData writeToFile:sFilePath atomically:YES];
            }
    }];

    [sPostDataTask resume];
}

```
## HTTP Response attachment
```obj-c

#pragma mark - WKNavigationDelegate

- (void)webView:(WKWebView *)webView decidePolicyForNavigationResponse:(WKNavigationResponse *)navigationResponse decisionHandler:(void (^)(WKNavigationResponsePolicy))decisionHandler
{
    BOOL            sIsFileDownload         = NO;
    NSURL           *sURL                   = navigationResponse.response.URL;
    NSString        *sMIMEType              = navigationResponse.response.MIMEType;
    NSString        *sFileName              = navigationResponse.response.suggestedFilename;
    NSDictionary    *sHeaders               = ((NSHTTPURLResponse *)navigationResponse.response).allHeaderFields;
    NSString        *sContentDisposition    = sHeaders[@"Content-Disposition"];
    BOOL            sIsAttachment           = [sContentDisposition containsString:@"attachment"];
    BOOL            sIsSpreadSheetType      = [sMIMEType containsString:@"application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"];

    if (sIsSpreadSheetType || sIsAttachment)
    {
        NSURLRequest *sRequest = [NSURLRequest requestWithURL:sURL];
        [self downloadTaskFromRequest:sRequest fileName:sFileName];

        sIsFileDownload = YES;
    }
    else
    {
        sIsFileDownload = NO;
    }

    decisionHandler(sIsFileDownload ? WKNavigationResponsePolicyCancel : WKNavigationResponsePolicyAllow);
}

```
