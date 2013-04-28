BoxSDK: Box API V2 iOS SDK
==========================

This SDK provides access to the [Box V2 API](https://developers.box.com/docs/).
It currently supports file and folder operations.

## Add to your project

The easiest way to add the Box SDK to your project is as a dependent XCode project.

1. Clone this repository into your project's directory. You can use git submodules
   if you want.
2. Open your project in XCode.
3. Drag BoxSDK.xcodeproj into the root of your project explorer.<br />![Dependent project](http://box.github.io/box-ios-sdk-private/readme-images/dependent-project.png)

4. Add the BoxSDK project as a target dependency.<br />![Target dependency](http://box.github.io/box-ios-sdk-private/readme-images/target-dependency.png)

5. Link with libBoxSDK.a<br />![Link with binary](http://box.github.io/box-ios-sdk-private/readme-images/link-with-binary.png)

6. Add the `-ObjC` linker flag. This is needed to load categories defined in the SDK.<br />![Add linker flag](http://box.github.io/box-ios-sdk-private/readme-images/linker-flag.png)

7. `#import <BoxSDK/BoxSDK.h>`

## Quickstart

### Configure

Set your client ID and client secret on the SDK client:

```objc
[BoxSDK sharedSDK].OAuth2Session.clientID = @"YOUR_CLIENT_ID";
[BoxSDK sharedSDK].OAuth2Session.clientSecret = @"YOUR_CLIENT_SECRET";
```

The SDK requires your app to register a custom URL scheme in order to receive
an OAuth2 authorization code. In your `Info.plist`, register the following URL
scheme:

```
boxsdk-YOUR_CLIENT_ID
```

### Authenticate
To authenticate your app with Box, you need to use OAuth2. The authorization flow
happens in a `UIWebView`. To get started, you can present the sample web view the
SDK provides:

```objc
UIViewController *authorizationController = [[BoxAuthorizationViewController alloc] initWithOAuth2Session:[BoxSDK sharedSDK].OAuth2Session];
[self presentViewController:authorizationController animated:YES completion:nil];
```

On successful authentication, your app will receive an "open in" request using
the custom URL scheme you registered earlier. In your app delegate:

```objc
- (BOOL)application:(UIApplication *)application
            openURL:(NSURL *)url
  sourceApplication:(NSString*)sourceApplication
         annotation:(id)annotation
{
    [[BoxSDK sharedSDK].OAuth2Session performAuthorizationCodeGrantWithReceivedURL:url];
    return YES;
}
```

You can listen to notifications on `[BoxSDK sharedSDK].OAuth2Session` to be notified
when a user becomes successfully authenticated.

### Making API calls

All SDK API calls are asynchronous. They are scheduled by the SDK on an `NSOperationQueue`.
To be notified of API responses and errors, pass blocks to the SDK API call methods. These
blocks are triggered once the API response has been received by the SDK.

**Note**: callbacks are not triggered on the main thread. Wrap updates to your app's
UI in a `dispatch_sync` block on the main thread.

#### Get a folder's children

```objc
BoxCollectionBlock success = ^(BoxCollection *collection)
{
  // grab items from the collection, use the collection as a data source
  // for a table view, etc.
};

BoxAPIJSONFailureBlock failure = ^(NSURLRequest *request, NSHTTPURLResponse *response, NSError *error, NSDictionary *JSONDictionary)
{
  // handle errors
};

[[BoxSDK sharedSDK].foldersManager folderItemsWithID:folderID requestBuilder:nil success:success failure:failure];
```

#### Get a file's information

```objc
BoxFileBlock success = ^(BoxFile *file)
{
  // manipulate the BoxFile.
};

BoxAPIJSONFailureBlock failure = ^(NSURLRequest *request, NSHTTPURLResponse *response, NSError *error, NSDictionary *JSONDictionary)
{
  // handle errors
};

[[BoxSDK sharedSDK].filesManager fileInfoWithID:folderID requestBuilder:nil success:success failure:failure];
```

#### Edit an item's information

To send data via the API, use a request builder. If we wish to move a file and change its
name:

```objc
BoxFileBlock success = ^(BoxFile *file)
{
  // manipulate the BoxFile.
};

BoxAPIJSONFailureBlock failure = ^(NSURLRequest *request, NSHTTPURLResponse *response, NSError *error, NSDictionary *JSONDictionary)
{
  // handle errors
};

BoxFilesRequestBuilder *builder = [BoxFilesRequestBuilder alloc] init];
builder.name = @"My awesome file.txt"
builder.parentID = BoxAPIFolderIDRoot;

[[BoxSDK sharedSDK].filesManager editFileWithID:folderID requestBuilder:builder success:success failure:failure];
```

#### Upload a new file

```objc
BoxFileBlock fileBlock = ^(BoxFile *file)
{
  // manipulate resulting BoxFile
};

BoxAPIJSONFailureBlock failureBlock = ^(NSURLRequest *request, NSHTTPURLResponse *response, NSError *error, NSDictionary *JSONDictionary)
{
  // handle failed upload
};

BoxAPIMultipartProgressBlock progressBlock = ^(unsigned long long totalBytes, unsigned long long bytesSent)
{
  // indicate progress of upload
};

BoxFilesRequestBuilder *builder = [[BoxFilesRequestBuilder alloc] init];
builder.name = @"Logo_Box_Blue_Whitebg_480x480.jpg";
builder.parentID = folderID;

NSString *path = [[NSBundle mainBundle] pathForResource:@"Logo_Box_Blue_Whitebg_480x480.jpg" ofType:nil];
NSInputStream *inputStream = [NSInputStream inputStreamWithFileAtPath:path];
NSDictionary *fileAttributes = [[NSFileManager defaultManager] attributesOfItemAtPath:path error:nil];
long long contentLength = [[fileAttributes objectForKey:NSFileSize] longLongValue];


NSData *data = [NSData dataWithContentsOfFile:[[NSBundle mainBundle] pathForResource:@"Logo_Box_Blue_Whitebg_480x480" ofType:@"jpg"]];

BoxAPIMultipartToJSONOperation *operation = [[BoxSDK sharedSDK].filesManager uploadFileWithInputStream:inputStream contentLength:contentLength MIMEType:nil requestBuilder:builder success:fileBlock failure:failureBlock progress:progressBlock];
```

#### Download a file

```objc
NSOutputStream *outputStream = [NSOutputStream outputStreamToFileAtPath:path append:NO];

BoxDownloadSuccessBlock successBlock = ^(NSString *downloadedFileID, long long expectedContentLength)
{
  // handle download, preview download, etc.
};

BoxDownloadFailureBlock failureBlock = ^(NSURLRequest *request, NSHTTPURLResponse *response, NSError *error)
{
  // handle download failure
};

BoxAPIDataProgressBlock progressBlock = ^(long long expectedTotalBytes, unsigned long long bytesReceived)
{
  // display progress
};

[[BoxSDK sharedSDK].filesManager downloadFileWithID:fileID outputStream:outputStream requestBuilder:nil success:successBlock failure:failureBlock progress:progressBlock];
```

## Tests

This SDK contains unit tests that are runnable with `./bin/test.sh`.

Pull requests will not be accepted unless they include test coverage.

## Documentation

Documentation for this SDK is generated using [appledoc](http://gentlebytes.com/appledoc/).
And can be generated by running `./bin/generate-documentation.sh`.

Documentation is hosted on this repo's github page.

Pull requests will not be accepted unless they include documentation.

## Known issues

* There is no support for manipulating files in the trash.
* Only the files and folders endpoints are supported.

