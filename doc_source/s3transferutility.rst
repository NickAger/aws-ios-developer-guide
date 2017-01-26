.. Copyright 2010-2017 Amazon.com, Inc. or its affiliates. All Rights Reserved.

   This work is licensed under a Creative Commons Attribution-NonCommercial-ShareAlike 4.0
   International License (the "License"). You may not use this file except in compliance with the
   License. A copy of the License is located at http://creativecommons.org/licenses/by-nc-sa/4.0/.

   This file is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND,
   either express or implied. See the License for the specific language governing permissions and
   limitations under the License.

Amazon S3 Transfer Utility for iOS
#########################################

`Amazon S3 Transfer Manager for iOS <http://docs.aws.amazon.com/mobile/sdkforios/developerguide/s3transfermanager.html#create-the-s3-transfermanager-client>`_ simplifies data transfer between your
iOS app and Amazon S3. In addition to the S3 Transfer Manager, there is an easier and more powerful
way to transfer data between your iOS app and Amazon S3, Amazon S3 Transfer Utility for iOS. This topic
shows you how to get started with the new Amazon S3 Transfer Utility.

The Amazon S3 Transfer Utility offers two main advantages over the S3 Transfer Manager:

- Ability to continue transferring data in the background

- An API to upload binary data without first requiring it be saved as a file. The S3 Transfer Manager requires you to save data to a file before passing it to Transfer Manager.

Setting Up the S3 Transfer Utility
==================================

To set up the S3 Transfer Utility in your app, you must set up Amazon Cognito Identity to
authenticate calls to AWS from your app then configure your application delegate.

Setting Up Amazon Cognito Identity
---------------------------

First, you need to set up Amazon Cognito Identity.

Import the following header in your application delegate class.

    .. container:: option

        Swift
            .. code-block:: swift

                import AWSS3


        Objective-C
            .. code-block:: objc

                #import <AWSS3/AWSS3.h>

Set up ``AWSCognitoCredentialsProvider`` in the ``- application:didFinishLaunchingWithOptions:`` application delegate.

    .. container:: option

        Swift
            .. code-block:: swift

                func application(_ application: UIApplication,
                             didFinishLaunchingWithOptions launchOptions:
                             [ UIApplicationLaunchOptionsKey: Any]?) -> Bool {
                    let credentialProvider = AWSCognitoCredentialsProvider(regionType: .USEast1, identityPoolId: "YourIdentityPoolId")
                    let configuration = AWSServiceConfiguration(region:.USEast1, credentialsProvider:credentialProvider)
                    AWSServiceManager.default().defaultServiceConfiguration = configuration
                    return true
                }

        Objective-C
            .. code-block:: objc

                - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
                AWSCognitoCredentialsProvider *credentialsProvider = [[AWSCognitoCredentialsProvider alloc]
                initWithRegionType:AWSRegionUSEast1 identityPoolId:@"YourIdentityPoolId"];
                    AWSServiceConfiguration *configuration = [[AWSServiceConfiguration alloc] initWithRegion:AWSRegionUSEast1
                        credentialsProvider:credentialsProvider];
                    AWSServiceManager.defaultServiceManager.defaultServiceConfiguration = configuration;

                    return YES;
                }

Refer to `Set Up the SDK for iOS <http://docs.aws.amazon.com/mobile/sdkforios/developerguide/setup.html>`_
and `Amazon Cognito Identity <http://docs.aws.amazon.com/mobile/sdkforios/developerguide/cognito-auth.html>`_ sections
of `iOS Developer Guide <http://docs.aws.amazon.com/mobile/sdkforios/developerguide/>`_ for more details.

Configuring the Application Delegate
------------------------------------

The Transfer Utility for iOS uses the background transfer feature in iOS to continue data transfers even when your app isn't
running.

Call the following class method in your ``- application:handleEventsForBackgroundURLSession:completionHandler:``
application delegate so that iOS can tell the Transfer Utility when the transfer finishes while the app is not suspended.

    .. container:: option

        Swift
            .. code-block:: swift

                func application(_ application: UIApplication, handleEventsForBackgroundURLSession identifier: String, completionHandler: @escaping () -> Void) {
                    // Store the completion handler. 
                    AWSS3TransferUtility.interceptApplication(application, handleEventsForBackgroundURLSession: identifier, completionHandler: completionHandler)
                }


        Objective-C
            .. code-block:: objc


                - (void)application:(UIApplication *)application handleEventsForBackgroundURLSession:(NSString *)identifier
                completionHandler:(void (^)())completionHandler {
                    /* Store the completion handler.*/
                    [AWSS3TransferUtility interceptApplication:application handleEventsForBackgroundURLSession:identifier completionHandler:completionHandler];
                }

Transferring Data
=================

After you set up the S3 Transfer Utility, you can upload or download files and binary data between S3 and your app.

Uploading a File
----------------

To upload a file call ``- uploadFile:bucket:key:contentType:expression:completionHander:`` on ``AWSS3TransferUtility``.

    .. container:: option

        Swift
            .. code-block:: swift

                let fileURL = // The file to upload
                let  transferUtility = AWSS3TransferUtility.default()
                transferUtility.uploadFile(fileURL,
                        bucket: S3BucketName,
                        key: S3UploadKeyName, 
                        contentType: "image/png",
                        expression: nil,
                        completionHandler: nil).continueWith {
                    (task) -> AnyObject! in if let error = task.error {
                        print("Error: \(error.localizedDescription)")
                    }

                    if let _ = task.result {
                        // Do something with uploadTask.
                    }
                    return nil;
                }

        Objective-C
            .. code-block:: objc

                NSURL *fileURL = // The file to upload.

                AWSS3TransferUtility *transferUtility = [AWSS3TransferUtility defaultS3TransferUtility];
                [[transferUtility uploadFile:fileURL
                                    bucket:@"YourBucketName"
                                    key:@"YourObjectKeyName"
                                    contentType:@"text/plain"
                                    expression:nil
                            completionHander:nil] continueWithBlock:^id(AWSTask *task) {
                    if (task.error) {
                        NSLog(@"Error: %@", task.error);
                    }
                    if (task.result) {
                        AWSS3TransferUtilityUploadTask *uploadTask = task.result;
                        // Do something with uploadTask.
                    }

                    return nil;
                }];

If you want to know when the upload completes, you can pass a completion handler block. You can also configure the upload
behavior by passing ``AWSS3TransferUtilityUploadExpression``. For example, you can add a upload progress feedback block to
the expression object. Here is a code snippet containing the completion handler and upload progress feedback:

    .. container:: option

        Swift
            .. code-block:: swift

                let  transferUtility = AWSS3TransferUtility.default()
                transferUtility.uploadFile(fileURL,
                            bucket: S3BucketName,
                            key: S3UploadKeyName,
                            contentType: "image/png",
                            expression: expression,
                            completionHandler).continueWith { (task) -> AnyObject! in
                    if let error = task.error {
                        print("Error: \(error.localizedDescription)")
                    }
                    if let _ = task.result {
                        // Do something with uploadTask.
                    }

                    return nil;
                }


        Objective-C
            .. code-block:: objc

                NSURL *fileURL = // The file to upload.

                AWSS3TransferUtilityUploadExpression *expression = [AWSS3TransferUtilityUploadExpression new];
                expression.progressBlock = ^(AWSS3TransferUtilityTask *task, NSProgress *progress) {
                    dispatch_async(dispatch_get_main_queue(), ^{
                        // Do something e.g. Update a progress bar.
                    });
                };

                AWSS3TransferUtilityUploadCompletionHandlerBlock completionHandler = ^(AWSS3TransferUtilityUploadTask *task, NSError *error) {
                    dispatch_async(dispatch_get_main_queue(), ^{
                        // Do something e.g. Alert a user for transfer completion.
                        // On failed uploads, `error` contains the error object.
                    });
                };

                AWSS3TransferUtility *transferUtility = [AWSS3TransferUtility defaultS3TransferUtility];
                [[transferUtility uploadFile:fileURL
                                bucket:@"YourBucketName"
                                key:@"YourObjectKeyName"
                                contentType:@"text/plain"
                                expression:expression
                        completionHander:completionHandler] continueWithBlock:^id(AWSTask *task) {
                    if (task.error) {
                        NSLog(@"Error: %@", task.error);
                    }
                    if (task.result) {
                        AWSS3TransferUtilityUploadTask *uploadTask = task.result;
                        // Do something with uploadTask.
                    }

                    return nil;
                }];

Uploading Binary Data
---------------------

To upload an instance of ``NSData`` call ``- uploadData:bucket:key:contentType:expression:completionHander:``

    .. container:: option

        Swift
            .. code-block:: swift

                let data = // The data to upload
                let expression = AWSS3TransferUtilityUploadExpression()
                expression.progressBlock = {(task, progress) in DispatchQueue.main.async(execute: {
                        // Do something e.g. Update a progress bar.
                    })
                }
                let completionHandler = { (task, error) -> Void in
                    DispatchQueue.main.async(execute: {
                        // Do something e.g. Alert a user for transfer completion.
                        // On failed uploads, `error` contains the error object.
                    })
                }

                let  transferUtility = AWSS3TransferUtility.default()

                transferUtility.uploadData(data,
                            bucket: S3BucketName,
                            key: S3UploadKeyName,
                            contentType: "image/png",
                            xpression: expression,
                            completionHandler: completionHandler).continueWith { (task) -> AnyObject! in
                    if let error = task.error {
                        print("Error: \(error.localizedDescription)")
                    }
 
                    if let _ = task.result {
                        // Do something with uploadTask.
                    }

                    return nil;
                }


        Objective-C
            .. code-block:: objc

                NSData *dataToUpload = // The data to upload.

                AWSS3TransferUtilityUploadExpression *expression = [AWSS3TransferUtilityUploadExpression new];
                expression.progressBlock = ^(AWSS3TransferUtilityTask *task, NSProgress *progress) {
                    dispatch_async(dispatch_get_main_queue(), ^{
                        // Do something e.g. Update a progress bar.
                    });
                };


                AWSS3TransferUtilityUploadCompletionHandlerBlock completionHandler = ^(AWSS3TransferUtilityUploadTask *task, NSError *error) {
                    dispatch_async(dispatch_get_main_queue(), ^{
                        // Do something e.g. Alert a user for transfer completion.
                        // On failed uploads, `error` contains the error object.
                    });
                };

                AWSS3TransferUtility *transferUtility = [AWSS3TransferUtility defaultS3TransferUtility];
                [[transferUtility uploadData:dataToUpload
                                bucket:@"YourBucketName"
                                key:@"YourObjectKeyName"
                                contentType:@"text/plain"
                                expression:expression
                        completionHander:completionHandler] continueWithBlock:^id(AWSTask *task) {
                    if (task.error) {
                        NSLog(@"Error: %@", task.error);
                    }
                    if (task.result) {
                        AWSS3TransferUtilityUploadTask *uploadTask = task.result;
                        // Do something with uploadTask.
                    }

                    return nil;
                }];

Note that this method saves the data as a file in a temporary directory. The next time ``AWSS3TransferUtility`` is
initialized, the expired temporary files are cleaned up. If you upload many large objects to an Amazon S3 bucket in a short
period of time, it's better to use the upload file method then manually purge the unnecessary temporary files as early as
possible for more efficient use of disk space.

Downloading to a File
---------------------

Here are code snippets you can use for downloading to a file.

    .. container:: option

        Swift
            .. code-block:: swift

                let fileURL = // The file URL of the download destination.
                let expression = AWSS3TransferUtilityDownloadExpression ()
                expression.progressBlock = {(task, progress) in
                    DispatchQueue.main.async(execute: {
                        // Do something e.g. Update a progress bar.
                    }
                }
                let completionHandler = { (task, location, data, error) -> Void in
                    DispatchQueue.main.async(execute: {
                        // Do something e.g. Alert a user for transfer completion.
                        // On successful downloads, `location` contains the S3 object file URL.
                        // On failed downloads, `error` contains the error object.
                    })
                }
                let  transferUtility = AWSS3TransferUtility.default()
                transferUtility.download(
                        to: fileURL
                        bucket: S3BucketName,
                        key: S3DownloadKeyName,
                        expression: expression,
                        completionHander: completionHandler
                ).continueWith {
                    (task) -> AnyObject! in if let error = task.error {
                        print("Error: \(error.localizedDescription)")
                    }

                    if let _ = task.result {
                        // Do something with downloadTask.
                    }
                    return nil;
                }

        Objective-C
            .. code-block:: objc

                NSURL *fileURL = ...; // The file URL of the download destination.

                AWSS3TransferUtilityDownloadExpression *expression = [AWSS3TransferUtilityDownloadExpression new];
                expression.progressBlock = ^(AWSS3TransferUtilityTask *task, NSProgress *progress) {
                    dispatch_async(dispatch_get_main_queue(), ^{
                        // Do something e.g. Update a progress bar.
                    });
                };

                AWSS3TransferUtilityDownloadCompletionHandlerBlock completionHandler = ^(AWSS3TransferUtilityDownloadTask *task, NSURL *location, NSData *data, NSError *error) {
                    dispatch_async(dispatch_get_main_queue(), ^{
                        // Do something e.g. Alert a user for transfer completion.
                        // On successful downloads, `location` contains the S3 object file URL.
                        // On failed downloads, `error` contains the error object.
                    });
                };

                AWSS3TransferUtility *transferUtility = [AWSS3TransferUtility defaultS3TransferUtility];
                [[transferUtility downloadToURL:nil
                                bucket:S3BucketName
                                key:S3DownloadKeyName
                                expression:expression
                        completionHander:completionHandler] continueWithBlock:^id(AWSTask *task) {
                    if (task.error) {
                        NSLog(@"Error: %@", task.error);
                    }
                    if (task.result) {
                        AWSS3TransferUtilityDownloadTask *downloadTask = task.result;
                        // Do something with downloadTask.
                    }

                    return nil;
                }];

Downloading as Binary Data
--------------------------

Here are code snippets you can use for downloading binary data.

    .. container:: option

        Swift
            .. code-block:: swift

                let fileURL = // The file URL of the download destination.
                let expression = AWSS3TransferUtilityDownloadExpression ()
                expression.progressBlock = {
                    (task, progress) in DispatchQueue.main.async( execute: {
                        // Do something e.g. Update a progress bar.
                    })
                }
                let completionHandler = {
                    (task, location, data, error) -> Void in DispatchQueue.main.async( execute: {
                        // Do something e.g. Alert a user for transfer completion.
                        // On successful downloads, `location` contains the S3 object file URL.
                        // On failed downloads, `error` contains the error object.
                    })
                }
                let  transferUtility = AWSS3TransferUtility.default()
                transferUtility.downloadData(
                        fromBucket: S3BucketName,
                        key: S3DownloadKeyName,
                        expression: expression,
                        completionHander: completionHandler
                ).continueWith {
                    (task) -> AnyObject! in if let error = task.error {
                        print("Error: \(error.localizedDescription)")
                    }

                    if let _ = task.result {
                        // Do something with downloadTask.
                    }

                    return nil;
                }

        Objective-C
            .. code-block:: objc

                AWSS3TransferUtilityDownloadExpression *expression = [AWSS3TransferUtilityDownloadExpression new];
                expression.progressBlock = ^(AWSS3TransferUtilityTask *task, NSProgress *progress) {
                    dispatch_async(dispatch_get_main_queue(), ^{
                        // Do something e.g. Update a progress bar.
                    });
                };

                AWSS3TransferUtilityDownloadCompletionHandlerBlock completionHandler = ^(
                    AWSS3TransferUtilityDownloadTask *task, NSURL *location, NSData *data, NSError *error) {
                        dispatch_async(dispatch_get_main_queue(), ^{
                            // Do something e.g. Alert a user for transfer completion.
                            // On successful downloads, `data` contains the S3 object.
                            // On failed downloads, `error` contains the error object.
                        }
                    );
                };

                AWSS3TransferUtility *transferUtility = [AWSS3TransferUtility defaultS3TransferUtility];
                [[transferUtility downloadDataFromBucket:S3BucketName
                    key:S3DownloadKeyName
                    expression:expression
                    completionHander:completionHandler] continueWithBlock:^id(AWSTask *task) {
                        if (task.error) {
                            NSLog(@"Error: %@", task.error);
                        }
                        if (task.result) {
                            AWSS3TransferUtilityDownloadTask *downloadTask = task.result;
                            // Do something with downloadTask.
                        }

                        return nil;
                    }
                ];

Transferring in the Background
------------------------------

All uploads and downloads continue in the background whether your app is active or in the background. If iOS
terminates your app while transfers are ongoing, the system continues the transfers in the background then launches your app
after the transfers finish. If the user terminates the app while transfers are ongoing, those transfers stop.

You can't persist blocks on disk so you need to rewire the completion handler and progress feedback blocks when your app
relaunches. You should call ``- enumerateToAssignBlocksForUploadTask:downloadTask:`` on ``AWSS3TransferUtility`` to reassign
the blocks as needed. Here is an example of reassigning blocks.

    .. container:: option

        Swift
            .. code-block:: swift

                override func viewDidLoad() {
                super.viewDidLoad()

                ...

                let transferUtility = AWSS3TransferUtility.default()

                var uploadProgressBlock: AWSS3TransferUtilityProgressBlock? = {(task: AWSS3TransferUtilityTask, progress: Progress) in
                    DispatchQueue.main.async {
                        // Handle progress feedback, e.g. update progress bar
                    }
                }
                var downloadProgressBlock: AWSS3TransferUtilityProgressBlock? = {
                    (task: AWSS3TransferUtilityTask, progress: Progress) in DispatchQueue.main.async {
                        // Handle progress feedback, e.g. update progress bar
                    }
                }
                var completionBlockUpload:AWSS3TransferUtilityUploadCompletionHandlerBlock? = {
                    (task, error) in DispatchQueue.main.async {
                        // perform some action on completed upload operation
                    }
                }
                var completionBlockDownload:AWSS3TransferUtilityDownloadCompletionHandlerBlock? = {
                    (task, url, data, error) in DispatchQueue.main.async {
                        // perform some action on completed download operation
                    }
                }

                transferUtility.enumerateToAssignBlocks(forUploadTask: {
                    (task, progress, completion) -> Void in
                        let progressPointer = AutoreleasingUnsafeMutablePointer<AWSS3TransferUtilityProgressBlock?>(& uploadProgressBlock)
                    let completionPointer = AutoreleasingUnsafeMutablePointer<AWSS3TransferUtilityUploadCompletionHandlerBlock?>(&completionBlockUpload)
                    // Reassign your progress feedback
                        progress?.pointee = progressPointer.pointee
                    // Reassign your completion handler.
                        completion?.pointee = completionPointer.pointee
                }, downloadTask: { (task, progress, completion) -> Void in
                    let progressPointer = AutoreleasingUnsafeMutablePointer<AWSS3TransferUtilityProgressBlock?>(&downloadProgressBlock)
                    let completionPointer = AutoreleasingUnsafeMutablePointer<AWSS3TransferUtilityDownloadCompletionHandlerBlock?>(&completionBlockDownload)

                    // Reassign your progress feedback
                    progress?.pointee = progressPointer.pointee

                    // Reassign your completion handler.
                    completion?.pointee = completionPointer.pointee
                })

        Objective-C
            .. code-block:: objc

                - (void)viewDidLoad {
                    [super viewDidLoad];

                    ...

                    AWSS3TransferUtility *transferUtility = [AWSS3TransferUtility defaultS3TransferUtility];
                    [transferUtility enumerateToAssignBlocksForUploadTask:^(
                        AWSS3TransferUtilityUploadTask *uploadTask,
                        __autoreleasing AWSS3TransferUtilityUploadProgressBlock *uploadProgressBlockReference,
                        __autoreleasing AWSS3TransferUtilityUploadCompletionHandlerBlock *completionHandlerReference
                    ) {
                        NSLog(@"%lu", (unsigned long)uploadTask.taskIdentifier);

                        // Use `uploadTask.taskIdentifier` to determine what blocks to assign.

                        *uploadProgressBlockReference = ...; // Reassign your progress feedback block.
                        *completionHandlerReference = ...; // Reassign your completion handler.
                    }
                    downloadTask:^(AWSS3TransferUtilityDownloadTask *downloadTask, __autoreleasing AWSS3TransferUtilityDownloadProgressBlock *downloadProgressBlockReference, __autoreleasing AWSS3TransferUtilityDownloadCompletionHandlerBlock *completionHandlerReference) {
                        NSLog(@"%lu", (unsigned long)downloadTask.taskIdentifier);

                            // Use `downloadTask.taskIdentifier` to determine what blocks to assign.
                       *downloadProgressBlockReference =  // Reassign your progress feedback block.
                       *completionHandlerReference = // Reassign your completion handler.
                    }];
                }

You receive ``AWSS3TransferUtilityUploadTask`` and ``AWSS3TransferUtilityDownloadTask`` when you initiate the upload and
download respectively.

- For upload

    .. container:: option

        Swift
            .. code-block:: swift

                if let uploadTask = task.result {
                    // Do something with uploadTask.
                }


        Objective-C
            .. code-block:: objc

                if (task.result) {
                    AWSS3TransferUtilityUploadTask *uploadTask = task.result;
                    // Do something with uploadTask.
                }

- For download

    .. container:: option

        Swift
            .. code-block:: swift

                if let downloadTask = task.result {
                    // Do something with downloadTask.
                }



        Objective-C
            .. code-block:: objc

                if (task.result) {
                    AWSS3TransferUtilityDownloadTask *downloadTask = task.result;
                    // Do something with downloadTask.
                }

They have a property called ``taskIdentifier``, which uniquely identifies the transfer task object within the Transfer Utility.
You may need to persist the identifier so that you can uniquely identify the upload/download task objects when rewiring the
blocks for app relaunch.

Managing Data Transfers
=======================

In order to suspend, resume, and cancel uploads and downloads, you need to retain references to ``AWSS3TransferUtilityUploadTask`` and ``AWSS3TransferUtilityDownloadTask``.

To manage data transfers call ``- suspend``, ``- resume``, and ``- cancel`` on ``AWSS3TransferUtilityUploadTask``
and ``AWSS3TransferUtilityDownloadTask``.

Limitations
===========

The S3 Transfer Utility generates Amazon S3 Pre-Signed URLs to use for background data transfer.
Using Amazon Cognito Identity, you receive AWS temporary credentials that are valid up to 60 minutes.
At the same time, generated S3 pre-signed URLs cannot last longer than that time. Because of this
limitation, the S3 Transfer Utility enforces 50 minute transfer timeouts, leaving a 10 minute
buffer before AWS temporary credentials are regenerated. After 50 minutes, you receive a transfer failure.

If you need to transfer data that cannot be transferred in under 50 minutes, use ``AWSS3`` instead.
