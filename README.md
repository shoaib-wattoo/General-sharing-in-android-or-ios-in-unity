## Objective

Main objective of this blog post is to give you an idea about general sharing in android and iOS in unity

## Step 1 : Introduction

Today we know many apps and games which provide general sharing of messages, scores, promotional images and much more. So here I’ll explain you how to accomplish this stuff using native code of java for android games and native code of objective-c for iOS games.

Using my few lines of code you will be able to share messages and photos in your games on any android device as well as on iOS device.

**Here we follow two parts of doc:**

- One for general sharing of android games
- Another for iOS games.

## Step 2 : Android

### 2.1 Share textual data on android device

Using the following lines of code you will be able to share textual data on any android device in your game.

```csharp
#if UNITY_ANDROID
// Create Refernece of AndroidJavaClass class for intent
AndroidJavaClass intentClass = newAndroidJavaClass("android.content.Intent");
// Create Refernece of AndroidJavaObject class intent
AndroidJavaObject intentObject = newAndroidJavaObject("android.content.Intent");
 
// Set action for intent
intentObject.Call("setAction", intentClass.GetStatic("ACTION_SEND"));
 
intentObject.Call<AndroidJavaObject>("setType", "text/plain");
 
//Set Subject of action
intentObject.Call("putExtra", intentClass.GetStatic("EXTRA_SUBJECT"), "Text Sharing ");
//Set title of action or intent
intentObject.Call("putExtra", intentClass.GetStatic("EXTRA_TITLE"), "Text Sharing ");
// Set actual data which you want to share
intentObject.Call("putExtra", intentClass.GetStatic("EXTRA_TEXT"), "Text Sharing Android Demo");
 
AndroidJavaClass unity = newAndroidJavaClass("com.unity3d.player.UnityPlayer");
AndroidJavaObject currentActivity = unity.GetStatic("currentActivity");
// Invoke android activity for passing intent to share data
currentActivity.Call("startActivity", intentObject);
#endif
```

Here **ACTION_SEND** is used to set action for intent which passes data to activity. **putExtra** parameter is used to pass extra information with intent.

### 2.2 Share images data on android device

Using the following lines of code you will be able to share images, screenshots and promotional images in android games.

```csharp
// Save your image on designate path
byte[] bytes = MyImage.EncodeToPNG();
string path = Application.persistentDataPath + "/MyImage.png";
File.WriteAllBytes(path, bytes);
 
AndroidJavaClass intentClass = new AndroidJavaClass("android.content.Intent");
AndroidJavaObject intentObject = new AndroidJavaObject("android.content.Intent");
 
intentObject.Call("setAction", intentClass.GetStatic("ACTION_SEND"));
intentObject.Call("setType", "image/*");
intentObject.Call("putExtra", intentClass.GetStatic("EXTRA_SUBJECT"), "Media Sharing ");
intentObject.Call("putExtra", intentClass.GetStatic("EXTRA_TITLE"), "Media Sharing ");
intentObject.Call("putExtra", intentClass.GetStatic("EXTRA_TEXT"), "Media Sharing Android Demo");
 
AndroidJavaClass uriClass = new AndroidJavaClass("android.net.Uri");
AndroidJavaClass fileClass = new AndroidJavaClass("java.io.File");
 
AndroidJavaObject fileObject = new AndroidJavaObject("java.io.File", path);// Set Image Path Here
AndroidJavaObject uriObject = uriClass.CallStatic("fromFile", fileObject);
 
// string uriPath =  uriObject.Call("getPath");
bool fileExist = fileObject.Call("exists");
Debug.Log("File exist : " + fileExist);
// Attach image to intent
if (fileExist)
intentObject.Call("putExtra", intentClass.GetStatic("EXTRA_STREAM"), uriObject);
AndroidJavaClass unity = new AndroidJavaClass("com.unity3d.player.UnityPlayer");
AndroidJavaObject currentActivity = unity.GetStatic("currentActivity");
currentActivity.Call("startActivity", intentObject);
```

## Step 3 : iOS

For iOS you have to create one objective-c class with extension **.mm** which contains two external methods one for **text sharing** and another for **image sharing** and put it in **Assets >> Plugin >> IOS** folder.

### 3.1 Share images in iOS games

```swift
// Method for image sharing
@implementation ViewController : UIViewController
-(void) shareMethod: (const char *) path Message : (const char *) shareMessage
{
    NSString *imagePath = [NSString stringWithUTF8String:path];
    
    //    UIImage *image      = [UIImage imageNamed:imagePath];
    UIImage *image = [UIImage imageWithContentsOfFile:imagePath];
    NSString *message   = [NSString stringWithUTF8String:shareMessage];
    NSArray *postItems  = @[message,image];
    
    UIActivityViewController *activityVc = [[UIActivityViewController alloc]initWithActivityItems:postItems applicationActivities:nil];
    
    if ( UI_USER_INTERFACE_IDIOM() == UIUserInterfaceIdiomPad && [activityVc respondsToSelector:@selector(popoverPresentationController)] ) {
        
        UIPopoverController *popup = [[UIPopoverController alloc] initWithContentViewController:activityVc];
        
        [popup presentPopoverFromRect:CGRectMake(self.view.frame.size.width/2, self.view.frame.size.height/4, 0, 0)
                               inView:[UIApplication sharedApplication].keyWindow.rootViewController.view permittedArrowDirections:UIPopoverArrowDirectionAny animated:YES];
    }
    else
        [[UIApplication sharedApplication].keyWindow.rootViewController presentViewController:activityVc animated:YES completion:nil];
    [activityVc release];
} 
 
// globally declare image sharing method
extern "C"{
    void _TAG_ShareTextWithImage(const char * path, const char * message){
        ViewController *vc = [[ViewController alloc] init];
        [vc shareMethod:path Message:message];
        [vc release];
    }
}
```

### 3.2 Share textual data on iOS games

```swift
//Method for only text sharing
-(void) shareOnlyTextMethod: (const char *) shareMessage
{
    
    NSString *message   = [NSString stringWithUTF8String:shareMessage];
    NSArray *postItems  = @[message];
    
    UIActivityViewController *activityVc = [[UIActivityViewController alloc] initWithActivityItems:postItems applicationActivities:nil];
    
    
    if (UI_USER_INTERFACE_IDIOM() == UIUserInterfaceIdiomPad &&  [activityVc respondsToSelector:@selector(popoverPresentationController)] ) {
        
        UIPopoverController *popup = [[UIPopoverController alloc] initWithContentViewController:activityVc];
        
        [popup presentPopoverFromRect:CGRectMake(self.view.frame.size.width/2, self.view.frame.size.height/4, 0, 0)
                               inView:[UIApplication sharedApplication].keyWindow.rootViewController.view permittedArrowDirections:UIPopoverArrowDirectionAny animated:YES];
    }
    else
        [[UIApplication sharedApplication].keyWindow.rootViewController presentViewController:activityVc animated:YES completion:nil];
    [activityVc release];
}
 
// Globally declare text sharing method
extern "C"{
    void _TAG_ShareSimpleText(const char * message){
        ViewController *vc = [[ViewController alloc] init];
        [vc shareOnlyTextMethod: message];
        [vc release];
    }
}
```

**In your script you have to import both methods as follows:**

```swift
// import external methods
 
[DllImport("__Internal")]
private static extern void sampleMethod (string iosPath, string message);
[DllImport("__Internal")]
private static extern void sampleTextMethod (string message);
```

Now, share textual data using one line of code in your script.

```swift
sampleTextMethod ("I Just Share Textual Info");
```

**Use the following code to share images in iOS games:**

```csharp
//Save Image
byte[] bytes = MyImage.EncodeToPNG();
string path = Application.persistentDataPath + "/MyImage.png";
File.WriteAllBytes(path, bytes);
 
string path_ =  "MyImage.png";
 
string shareMessage = "Wow I Just Scored 10 ";
sampleMethod (path, shareMessage);
```

Thus, this is the way in which we can share our data in our games in android as well as in iOS.
