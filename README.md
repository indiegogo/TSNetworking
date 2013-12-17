TSNetworking
============

Because I wanted to see how NSURLSession worked.

Warning: the success and error blocks are executed on whatever thread apple decides.
I suggest if you're doing UI changes in these blocks that you dispatch_async and get the main queue.
Warning for young players: never reference self inside a block, use this style to avoid retain cycles
    
    __weak typeof(self) weakSelf = self;
    TSNetworkSuccessBlock successBlock = ^(NSObject *resultObject, NSMutableURLRequest *request, NSURLResponse *response) {
        __strong typeof(weakSelf) strongSelf = weakSelf;
        [strongSelf sendAMessage];
    };
## Initialising:

    [[TSNetworking sharedSession] setBaseURLString:kBaseURLString];
	[[TSNetworking sharedSession] setBasicAuthUsername:nil withPassword:nil];
These settings last for the app's run lifetime

## Get:

	TSNetworkSuccessBlock successBlock = ^(NSObject *resultObject, NSMutableURLRequest *request, NSURLResponse *response) {
        NSLog(@"We got ourselves a: %@", resultObject);
    };
    
    TSNetworkErrorBlock errorBlock = ^(NSObject *resultObject, NSError *error, NSMutableURLRequest *request, NSURLResponse *response) {
        switch (error.code) {
            case 401:
                NSLog(@"Mannn");
                break;
                
            default:
                break;
        }
    };

    [[TSNetworking sharedSession] performDataTaskWithRelativePath:nil
                                                       withMethod:HTTP_METHOD_GET
                                                   withParameters:nil
                                                      withSuccess:successBlock
                                                        withError:errorBlock];

## Post:

    TSNetworkSuccessBlock successBlock = ^(NSObject *resultObject, NSMutableURLRequest *request, NSURLResponse *response) {
        // resultObject will probably be a JSON dictionary 
    };
    
    TSNetworkErrorBlock errorBlock = ^(NSObject *resultObject, NSError *error, NSMutableURLRequest *request, NSURLResponse *response) {
        NSLog(@"%@", error.localizedDescription); // tells you what the problem was, i.e. offline
    };
    
    [[TSNetworking sharedSession] performDataTaskWithRelativePath:nil
                                                       withMethod:HTTP_METHOD_POST
                                                   withParameters:@{@"key": @"value"}
                                                      withSuccess:successBlock
                                                        withError:errorBlock];

## Download:

    TSNetworkSuccessBlock successBlock = ^(NSObject *resultObject, NSMutableURLRequest *request, NSURLResponse *response) {
        // sweet, resultObject is a (local) NSURL for what we just downloaded
    };
    
    TSNetworkErrorBlock errorBlock = ^(NSObject *resultObject, NSError *error, NSMutableURLRequest *request, NSURLResponse *response) {
        //same old handling
    };
    
    TSNetworkDownloadTaskProgressBlock progressBlock = ^(int64_t bytesWritten, int64_t totalBytesWritten, int64_t totalBytesExpectedToWrite) {
        NSLog(@"Download written: %lld, total written: %lld, total expected: %lld", bytesWritten, totalBytesWritten, totalBytesExpectedToWrite);
    };
    
    [[TSNetworking backgroundSession] downloadFromFullPath:@"https://archive.org/download/1mbFile/1mb.mp4"
                                                    toPath:destinationPath
                                         withProgressBlock:progressBlock
                                               withSuccess:successBlock
                                                 withError:errorBlock];
                                                 
## Upload:

    NSString *sourcePath = [[NSBundle mainBundle] pathForResource:@"ourLord" ofType:@"jpg"];
    XCTAssertNotNil(sourcePath, @"Couldn't find local picture of our lord");
    
    TSNetworkSuccessBlock successBlock = ^(NSObject *resultObject, NSMutableURLRequest *request, NSURLResponse *response) {
        //resultObject is whatever the server responded with, could be JSON, HTML, whatever
    };
    
    TSNetworkErrorBlock errorBlock = ^(NSObject *resultObject, NSError *error, NSMutableURLRequest *request, NSURLResponse *response) {
        //same old handling
    };
    
    TSNetworkDownloadTaskProgressBlock progressBlock = ^(int64_t bytesWritten, int64_t totalBytesWritten, int64_t totalBytesExpectedToWrite) {
        NSLog(@"uploaded: %lld, total written: %lld, total expected: %lld", bytesWritten, totalBytesWritten, totalBytesExpectedToWrite);
    };
    
    [[TSNetworking backgroundSession] uploadFromFullPath:sourcePath
                                                  toPath:@"http://localhost:8080/upload"
                                       withProgressBlock:progressBlock
                                             withSuccess:successBlock
                                               withError:errorBlock];