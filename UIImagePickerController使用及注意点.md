UIImagePickerController简述：

UIImagePickerController 类是获取选择图片和视频的用户接口，我们可以用UIImagePickerController选择我们所需要的图片和视频。

注：UIImagePickerController不能够任意定制，也不可以继承生成子类。

##### 1.UIImagePickerController的属性

（1）sourceType

指定使用模式（照相机/相册／照片库）

typedef NS_ENUM(NSInteger, UIImagePickerControllerSourceType) {

  UIImagePickerControllerSourceTypePhotoLibrary,

  UIImagePickerControllerSourceTypeCamera,

  UIImagePickerControllerSourceTypeSavedPhotosAlbum

}

（2）showsCameraControls

设置拍照时的下方的工具栏是否显示，如果需要自定义拍摄界面，则可把该工具栏隐藏

（3）allowsEditing

设置当拍照完或在相册选完照片后，是否跳到编辑模式进行图片剪裁，showsCameraControls=YES时才有效果。

（4）cameraDevice

判断设备是否支持前置摄像头／后置摄像头

typedef NS_ENUM(NSInteger, UIImagePickerControllerCameraDevice) {

  UIImagePickerControllerCameraDeviceRear,

  UIImagePickerControllerCameraDeviceFront

}

（5）cameraFlashMode

设置闪光灯模式

typedef NS_ENUM(NSInteger, UIImagePickerControllerCameraFlashMode) {

  UIImagePickerControllerCameraFlashModeOff = -1,

  UIImagePickerControllerCameraFlashModeAuto = 0,

  UIImagePickerControllerCameraFlashModeOn  = 1

}

（6）mediaTypes

设置相机支持的类型，拍照和录像

\+ (NSArray *)availableMediaTypesForSourceType:(UIImagePickerControllerSourceType)sourceType
一共有三个可选的代理方法UIImagePickerControllerDelegate 
– imagePickerController:didFinishPickingMediaWithInfo:  
– imagePickerControllerDidCancel:  
– imagePickerController:didFinishPickingImage:editingInfo: 

\- (void)imagePickerController:(UIImagePickerController *)picker didFinishPickingMediaWithInfo:(NSDictionary *)info
info中包括选取的照片，视频的主要信息
NSString *const UIImagePickerControllerMediaType;     选取的类型 public.image public.movie
NSString *const UIImagePickerControllerOriginalImage;  修改前的UIImage object.
NSString *const UIImagePickerControllerEditedImage;   修改后的UIImage object.
NSString *const UIImagePickerControllerCropRect;      原始图片的尺寸NSValue object containing a CGRect data type
NSString *const UIImagePickerControllerMediaURL;     视频在文件系统中 的 NSURL地址

（7）cameraViewTransform

设置拍摄时屏幕的view的transform属性，可以实现旋转，缩放功能

##### 2.UIImagePickerController回调处理

（1）成功获得相片或视频后的回调

\- (void)imagePickerController:(UIImagePickerController *)picker didFinishPickingMediaWithInfo:(NSDictionary *)info；

（2）取消照相机的回调

\- (void)imagePickerControllerDidCancel:(UIImagePickerController *)picker；

（3）保存照片成功后的回调

\- (void)image:(UIImage *)image didFinishSavingWithError:(NSError *)error *contextInfo:(void \*)contextInfo；*

##### 3.**检查硬件是否安装有摄像头或者允许操作相册**

（1） 判断设备是否有摄像头

-(BOOL)isCameraAvailable{

  return [UIImagePickerController isSourceTypeAvailable:UIImagePickerControllerSourceTypeCamera];

}

（2）前面的摄像头是否可用

-(BOOL)isFrontCameraAvailable{

  return [UIImagePickerController isCameraDeviceAvailable:UIImagePickerControllerCameraDeviceFront];

}

（3）后面的摄像头是否可用

-(BOOL)isRearCameraAvailable{

  return [UIImagePickerController isCameraDeviceAvailable:UIImagePickerControllerCameraDeviceRear];

}

（4）相册是否可用

-(BOOL)isPhotoLibraryAvailable{

  return [UIImagePickerController isSourceTypeAvailable:UIImagePickerControllerSourceTypePhotoLibrary];

}

注意：1.设置 allowsEditing 属性时，在成功获得相片回调中，取info中的图片的key是不同的。

​      2.在访问相机和相册时，需要在项目的info.plist进行配置访问权限，不授权则无法使用，APP会crash

​     Privacy - Photo Library Usage Description ：相册权限

​     Privacy - Camera Usage Description：相机权限