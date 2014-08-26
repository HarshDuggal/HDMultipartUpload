HDMultipartUpload
=================


Step 1:
     UIImage is converted into in NSData.
     
Step 2:
     Data is divided into chunks.
     
Step 3:
     Chunks are send to server sequentially.
     
Step 4:     
     Check Server response.
           Case 1- if failed to upload same chunk is resend.
           Case 2- else if successful next chunk is sent server.
 
 
 
 
 Usage Code
 ============
 Import by draging the
 HDMultiPartImageUpload.h and HDMultiPartImageUpload.m
 to your project file
 
 Add the following code to your class and start uploading.

``` 
#import HDMultiPartImageUpload.h 

@implementation MyDemoVC
@property(nonatomic,strong) NSString * filePath
@end 

@implementation MyDemoVC 
 
-(void)demoupload 
{
    NSArray *paths = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES);
    
    NSString *documentsDirectory = [paths objectAtIndex:0]; 
    
    documentsDirectory = [NSString stringWithFormat:@"%@/ProfilePic/",documentsDirectory]; // navigate the image dicrectory
    
    NSFileManager*fmanager = [NSFileManager defaultManager]; 
    
    if(![fmanager fileExistsAtPath:documentsDirectory]) // create file path if not there
    {
        [fmanager createDirectoryAtPath:documentsDirectory withIntermediateDirectories:YES attributes:nil error:nil];
        
    }
    
    NSString * filePath =  [NSString stringWithFormat:@"%@",documentsDirectory];
    
    NSMutableDictionary *postParam = [[NSMutableDictionary alloc]init];
    
    [postParam addEntriesFromDictionary:[self demoPostDict]];
    
    HDMultiPartImageUpload *obj = [[HDMultiPartImageUpload alloc]init]; //intilaize the object and set all the required parameters.
    
    obj.oneChunkSize = 1024 *10;
    
    obj.selectedImageType = eImageTypePNG;
    
    obj.imageFilePath =filePath;
    
    obj.uploadURLString = @"http://example.com/upload";
    
    obj.postParametersDict = postParam;
    
    [obj startUploadImagesToServer];
}

-(NSMutableDictionary*)demoPostDict // Create a dictionary of the parameters needed to be passed at server

{
    NSMutableDictionary *param = [[NSMutableDictionary alloc]init];
    
    #warning - These key values in post dictionary varies according to the server implementation----
    
    UIImage *imageTobeUploaded = [UIImage imageWithContentsOfFile:self.imageFilePath];
    
    NSData *imageData;
    
    NSString *fileType;
    
    if (self.selectedImageType == eImageTypeJPG)
    {
        
        imageData = UIImageJPEGRepresentation(imageTobeUploaded, 1.0);
        
        fileType = @"image/jpg";
        
    }
    
    else if (self.selectedImageType == eImageTypePNG) 
    {
        
        imageData = UIImagePNGRepresentation(imageTobeUploaded);
        
        fileType = @"image/png";
        
    }
    
    NSUInteger totalFileSize = [imageData length];
    
    //    int totalChunks = ceil(totalFileSize/oneChunkSize);
    
    int totalChunks = round((totalFileSize/self.oneChunkSize)+0.5);//round-off to nearest  largest valua 1.01 is considered as 2
    
    // Create your Post parameter dict according to server
    NSString* originalFilename = @"tmpImageToUpload.png";//uniqueFileName;
    
    //Creating a unique file to upload to server
    NSString *prefixString = @"Album";
    //    This method generates a new string each time it is invoked, so it also uses a counter to guarantee that strings created from the same process are unique.
    NSString *guid = [[NSProcessInfo processInfo] globallyUniqueString] ;//
    NSString *uniqueFileName = [NSString stringWithFormat:@"%@_%@", prefixString, guid];
    
    //Add key values your post param Dict
    [param setObject:uniqueFileName
              forKey:@"uniqueFilename"];
    [param setObject:[NSString stringWithFormat:@"%lu",(unsigned long)totalFileSize]
              forKey:@"totalFileSize"];
    [param setObject:@"0" forKey:@"chunk"];
    [param setObject:[NSString stringWithFormat:@"%d",totalChunks]
              forKey:@"chunks"];
    [param setObject:fileType
              forKey:@"fileType"];
    [param setObject:originalFilename
              forKey:@"originalFilename"];
    
    //- #warning - These key values in post dictionary varies according to the server implementation----
    return param;
    
}
```



License
=============
AFNetworking is available under the MIT license. See the LICENSE file for more info.
