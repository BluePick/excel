# excel
Here is best sample project link
- (BOOL) drawImage:(int)index
{
    if (fltYPosToStartDwawing+320>=pageSize.height) {
        return NO;
    }
    BOOL __block moveForward = NO;
    NSString *filepath = [[self.arrProjectNoteDetails objectAtIndexedSubscript:index] valueForKey:@"Photo_Path"];
    UIImage * __block demoImage;
    if ([filepath hasPrefix:@"/Documents/MyImages"]) {
        demoImage = [UIImage imageWithContentsOfFile:[NSString stringWithFormat:@"%@%@",NSHomeDirectory(),filepath]];
        moveForward = YES;
    }
    else{
        ALAssetsLibrary *assetLibrary = [[ALAssetsLibrary alloc] init];
        [assetLibrary assetForURL:[NSURL URLWithString:filepath] resultBlock:^(ALAsset *asset)
        {
            if(asset)
            {
                demoImage = [UIImage imageWithCGImage:[[asset defaultRepresentation] fullResolutionImage]];
                moveForward = YES;
            }
            else if([filepath hasPrefix:@"assets-library"]){
                [assetLibrary enumerateGroupsWithTypes:ALAssetsGroupPhotoStream
                                            usingBlock:^(ALAssetsGroup group, BOOL stop)
                 {
                     [group enumerateAssetsWithOptions:NSEnumerationReverse usingBlock:^(ALAsset result, NSUInteger index, BOOL stop) {
                         if([result.defaultRepresentation.url isEqual:[NSURL URLWithString:filepath]])
                         {
                             demoImage = [UIImage imageWithCGImage:[[result defaultRepresentation] fullResolutionImage]];
                             moveForward = YES;
                         }
                     }];
                 }
                 failureBlock:^(NSError *error)
                 {
                     NSLog(@"Error: Cannot load asset from photo stream - %@", [error localizedDescription]);
                     
                 }];
            }
        }failureBlock:^(NSError *err)
         {
             NSLog(@"eror %@",err);
             moveForward = YES;
         }];
    }
    while (!moveForward) {
        [[NSRunLoop currentRunLoop] runUntilDate:[NSDate dateWithTimeIntervalSinceNow:0.1]];
    }
    CGSize temp = [demoImage size];
    float scale;
    if (temp.height>temp.width) {
        scale = 280/temp.height;
    }
    else{
        scale = 400/temp.width;
    }
    if (scale>1.0) {
        scale = 1.0;
    }
    float fltphotowidth=(temp.width*scale),fltphotoheight=(temp.height*scale);
    if (fltYPosToStartDwawing+fltphotoheight+40>=pageSize.height) {
        return NO;
    }
    
    NSString *strphotoname = [NSString stringWithFormat:@"%@ %@",[[self.arrProjectNoteDetails objectAtIndexedSubscript:index] valueForKey:@"Photo_Title"],[[AppDelegate objSharedAppDel] getTimeInStringWithComma:[[self.arrProjectNoteDetails objectAtIndex:index] valueForKey:@"Created"]]];
    
    UIFont *font = [UIFont systemFontOfSize:12];
    NSMutableParagraphStyle *paragraphStyle = [[NSParagraphStyle defaultParagraphStyle] mutableCopy];
    paragraphStyle.lineBreakMode = NSLineBreakByTruncatingMiddle;
    paragraphStyle.alignment = NSTextAlignmentLeft;
    NSDictionary *attributes = @{ NSFontAttributeName: font, NSParagraphStyleAttributeName: paragraphStyle };
    
    CGRect stringRenderingRect = CGRectMake(kBorderInset+15, fltYPosToStartDwawing, pageSize.width - 2*kBorderInset-30,20);
    [strphotoname drawInRect:stringRenderingRect withAttributes:attributes];
    
    CGSize sizetoResize = CGSizeMake((temp.width*scale), (temp.height*scale));
    demoImage = [[AppDelegate objSharedAppDel] imageByScalingAndCroppingForSourceImage:demoImage targetSize:sizetoResize];
    
    [demoImage drawInRect:CGRectMake((pageSize.width/2)-(fltphotowidth/2), fltYPosToStartDwawing+20, fltphotowidth, fltphotoheight)];
    fltYPosToStartDwawing+=fltphotoheight+30;
    [self drawLine:2];
    return YES;
}
