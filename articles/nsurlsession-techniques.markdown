---
layout: page
title: "NSURLSession Techniques"
date: 2014-03-03 18:34
comments: true
sharing: true
footer: true
---

## Introduction

In this article I'll offer some `NSURLSession` techniques to accomplish common networking functionality required in an app:

* **Data fetches**: Reading data from a remote URL to memory.
* **File downloads**: Downloading a file from a remote URL and saving it to disk.
* **Background downloads**: Downloading a (large) file from a remote URL and saving it to disk, in the background (i.e. even if your app is closed).

Here's the code: 

```objective-c
#import <Foundation/Foundation.h>

@interface Networker : NSObject

+ (void)fetchContentsOfURL:(NSURL *)url
                completion:(void (^)(NSData *data, NSError *error)) completionHandler;

+ (void)downloadFileAtURL:(NSURL *)url
               toLocation:(NSURL *)destinationURL
               completion:(void (^)(NSError *error)) completionHandler;
@end

@implementation Networker

+ (NSURLSession *)dataSession {
    static NSURLSession *session = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        session = [NSURLSession sessionWithConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration]];
    });
    return session;
}

+ (void)fetchContentsOfURL:(NSURL *)url
                completion:(void (^)(NSData *data, NSError *error)) completionHandler {
        
    NSURLSessionDataTask *dataTask =
    [[self dataSession] dataTaskWithURL:url
                      completionHandler:
     
     ^(NSData *data, NSURLResponse *response, NSError *error) {
         
         if (completionHandler == nil) return;
         
         if (error) {
             completionHandler(nil, error);
             return;
         }
         completionHandler(data, nil);
     }];
    
    [dataTask resume];
}

+ (void)downloadFileAtURL:(NSURL *)url
               toLocation:(NSURL *)destinationURL
               completion:(void (^)(NSError *error)) completionHandler {
    
    NSURLSessionDownloadTask *fileDownloadTask =
    [[self dataSession] downloadTaskWithRequest:[NSURLRequest requestWithURL:url]
                              completionHandler:
     
     ^(NSURL *location, NSURLResponse *response, NSError *error) {
         
         if (completionHandler == nil) return;
         
         if (error) {
             completionHandler(error);
             return;
         }
         
         NSError *fileError = nil;
         [[NSFileManager defaultManager] removeItemAtURL:destinationURL error:NULL];
         [[NSFileManager defaultManager] moveItemAtURL:location toURL:destinationURL error:&fileError];
         completionHandler(fileError);
     }];
    
    [fileDownloadTask resume];
}

@end

```

### Technique 1: Wrap NSURLSession

You can see that our networking code is inside a `Networker` class, isolated from other code, we didn't shove it inside, say, a `UIViewController` subclass.

You'll probably have more than one class in your app that needs data from the internet, but you don't want to replicate the same networking code in every single class that downloads or uploads data. What you should do is create a class that *wraps* `NSURLSession`, and whose responsibility is to be a networking hub that manages all your app's networking needs. This class will in turn be used by classes which require downloading or uploading, without them having to know or worry about `NSURLSession`. 

This is based on the assumption that all classes that use `Networker` are satisfied with an `NSURLSession` being configured in a single way. But if we have more complex requirements in which, for example, two different classes using two different remote APIs need to use two different timeout intervals for their requests, you'll be better off making making a `Networker` subclass for each use case, in which you adequately override `dataSession`.

### Technique 2: Provide the session via a static singleton method

`+[Networker dataSession]` is a static singleton method that returns a singleton `NSURLSession` object used for data fetches and file downloads. The more we centralise our networking, the easier it will be to control it. Advantages:

* You don't have to create and configure a `NSURLSession` object every time you want to create a `NSURLSessionTask`
* You centralise all the information relative to the active tasks in one place. For example, you can do:
```objective-c
[self dataSession] getTasksWithCompletionHandler:^(NSArray *dataTasks, NSArray *uploadTasks, NSArray *downloadTasks) {
    NSLog(@"Here are my data tasks: %@", dataTasks);
}
```
* You centralise control of all the `NSURLSessionTasks`, rather than having to make method calls on each individual one. For example:
```objective-c
[[self dataSession] invalidateAndCancel];
[[self dataSession] flushWithCompletionHandler:nil];

```

### Technique 3: Provide a completion block as a method parameter

This is partly a matter of style. When you're writing code that uses `Networker`, it tends to be more readable if the sequence of actions is clear.

This is also a matter of convenience, as the block will easily let you use variables available just in the scope of the method in which the networking was invoked. 

Let's do a quick example. Suppose we are inside a separate class. Now our block-based implementation:

```objective-c
- (void)userPressedButton {
    __block NSDate *timeButtonPressed = [NSDate date];
    [Networker fetchContentsOfURL:url completion:^(NSData *data, NSError *error) {
        [self useData:data forDate:timeButtonPressed];
    }];
}
```

With a delegate-based implementation: 

```objective-c
@interface SomeClass : NSObject <NetworkerDelegate>
@end

@implementation SomeClass {
    NSDate *timeButtonPressed;
}

- (void)userPressedButton {
    timeButtonPressed = [NSDate date];
    [Networker fetchContentsOfURL:url];
}
//56 lines further down...
- (void)networkerDidGetData:(NSData *)data error:(NSError *)error {
    [self useData:data forDate:timeButtonPressed];
}
```

The block-based alternative is shorter and more readable.

Also, remember that calling a block that's `nil` results in a crash, so we check we do have a `completionHandler` before calling it.


### Technique 4: Use class methods

There is little use for `Networker` to have instance variables if it is going to be a single, centralised networking hub.

The main advantage of class methods is that it avoids other objects the hassle about having to manage `Networker` instances.

Note that we are assuming that we are not required to report the progress of individual NSURLSessionTasks. If you're downloading potentially large and need to report the download progress to your users, then you need `Networker`

### Trying it out

We're going to put our `Networker` to the test, by using it on two free online APIs:
* We'll use the Open Weather Map API to get temperature data for Kathmandu.
* We'll use the Reddit API to download images.

Here's the idea:

1. Fetch the title of the top post on the front page of Reddit.
1. Use that as a search query to search for a picture on Reddit.
1. Download the image and display it. 

Open **ViewController.m** and add these lines:

```objective-c
#import "Networker.h"

@implementation ViewController

- (void)viewDidAppear:(BOOL)animated {
    [super viewDidAppear:animated];
    
    NSURL *redditFrontPageJSON = [NSURL URLWithString:@"http://www.reddit.com/r/pics.json"];
    
    [Networker fetchContentsOfURL:redditFrontPageJSON completion:^(NSData *data, NSError *error) {
        
        if (error == nil) {
            NSError *jsonParsingError = nil;
            NSDictionary *redditJSON = (NSDictionary *)[NSJSONSerialization JSONObjectWithData:data options:kNilOptions error:&jsonParsingError];
            
            if (jsonParsingError == nil) {
                NSArray *redditPosts = redditJSON[@"data"][@"children"];
                NSDictionary *topPost = [redditPosts firstObject];
                
                NSString *thumbnailURL = topPost[@"data"][@"thumbnail"];
                NSURL *documents = [[[NSFileManager defaultManager] URLsForDirectory:NSDocumentDirectory inDomains:NSUserDomainMask] firstObject];
                NSURL *fileURL = [documents URLByAppendingPathComponent:[thumbnailURL lastPathComponent]];
                
                [Networker downloadFileAtURL:[NSURL URLWithString:thumbnailURL] toLocation:fileURL completion:^(NSError *error) {
                    
                    if (error == nil) {
                        dispatch_async(dispatch_get_main_queue(), ^{
                            UIImage *image = [UIImage imageWithContentsOfFile:[fileURL path]];
                            UIImageView *imageView = [[UIImageView alloc] initWithImage:image];
                            [imageView setCenter:self.view.center];
                            [self.view addSubview:imageView];
                        });
                    }
                    else NSLog(@"Error downloading image: %@", error);
                }];
            }
            else NSLog(@"JSON parsing Error: %@", jsonParsingError);
        }
    }];
}

@end
```










