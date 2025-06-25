import UnityVersionedLink from '@site/src/components/UnityVersionedLink.js';

# Image Loading

Image Loading allows you to display images from the internet in your VRChat world. When a user visits your world, the image can be downloaded from the internet and used as a texture in your materials. Here are a few examples on how Image Loading can be used:

- Updating textures in your world without a re-upload.
- Creating a poster in your world and updating it for seasonal events or parties.
- Reusing the same texture in multiple worlds and updating them all at once.

The SDK includes an easy-to-use `ImageDownload` script, or you can make your own script with the new `VRCImageDownloader` object.

:::tip
You can [view our Image Loader example](/worlds/examples/image-loading) to get started quickly.
:::
## Before You Begin

There are a few Image Loader limits and parameters you should know:

- The maximum resolution is 2048 × 2048 pixels. Attempting to download larger images will result in an error.
- One image can be downloaded every five seconds.
  - If this limit is exceeded, images downloads are queued and downloaded in a random order.
  - This limit applies to your entire scene, regardless of the amount of VRCImageDownload components used.
- The URL must point directly at an image file. URL redirection is not allowed and will result in an error.
- Downloaded images are automatically interpreted as RGBA, RGB, or RG images.
  - For example, a grayscale image with an alpha channel is interpreted as an RG image.
- There is a limit of 1000 elements in the queue.
- Both the Input and Output buffers are limited to a maximum of 32MB, images exceeding these will result in an error.

And only certain domains are allowed. If a domain is not on the list, images will not download unless **Allow Untrusted URLs** has been enabled in the user's settings.


- DisBridge (`*.disbridge.com`)
- Dropbox (`dl.dropbox.com`,`dl.dropboxusercontent.com`)
- GitHub (`*.github.io`)
- ImageBam (`images4.imagebam.com`)
- ImgBB (`i.ibb.co`)
- imgbox (`images2.imgbox.com`)
- Imgur (`i.imgur.com`)
- Postimages (`i.postimg.cc`)
- Reddit (`i.redd.it`)
- Twitter (`pbs.twimg.com`)
- VRCDN (`*.vrcdn.cloud`)
- VRChat (`assets.vrchat.com`)
- Ytimg (`i.ytimg.com`)

## Memory Management

Downloaded images can take up a lot of memory. Once you're done using an image, you should dispose of it via Udon, which frees up memory for something else.

For example, if you download a new image and want to use it to replace another image you downloaded earlier, you should use the `Dispose` method documented below to remove the old image from memory. If you don't do this and keep downloading new images, visitors to your world may run out of memory and crash after spending enough time in your world!

## UdonGraph Nodes

### VRCImageDownloader

Use `VRCImageDownloader`'s constructor to create an image downloader, which can download image from the Internet during runtime.

#### DownloadImage

Downloads an image, and calls an event indicating success or failure (see 'Events' below).  
Returns an `IVRCImageDownload`, which can be used to track the progress of the download.

- **Instance**: The `ImageDownloader` component to download the image with.  
- **Url** : The `VRCURL` of the texture to download.  
- **Material** (optional): The Material to automatically apply the downloaded image to, as a main texture.
- **UdonBehavior** (optional): The `Udonbehavior` to send `VRCImageDownloader` events to. If `udonBehavior` is left empty in UdonGraph, the current UdonBehaviour will receive all events.
  - Note that UdonSharp will not receive any events unless `udonBehavior` is specified.
- **TextureInfo** (optional):  The `TextureInfo` object containing settings for the newly created texture.

#### Dispose

Cleans up the `VRCImageDownloader`. Frees up downloaded textures from memory.

Calling `Dispose` on a `VRCImageDownloader` invalidates the object, meaning it can't be used to download any new images.

**Notes on disposal and garbage collection:**

- Calling `Dispose` will invalidate the `VRCImageDownloader`, all of its associated `IVRCImageDownload` objects, and the textures associated with those downloads.
  - If you only want to dispose of a single download, call `Dispose` on the `IVRCImageDownload` object instead.
- Make sure to save the reference to your `VRCImageDownloader` as a variable to prevent it (and any downloaded texture) from randomly being garbage collected.

### TextureInfo

Contains settings to apply to a downloaded texture. 

- **GenerateMipmaps**: Enables Mipmap generation. (Default: `false`)
- **FilterMode**: Sets the `FilterMode` of the texture. (Default: `Trilinear`)
- **WrapModeU**: The `TextureWrapMode` along the U (horizontal) axis (Default: `Repeat`)
- **WrapModeV**: The `TextureWrapMode` along the V (vertical) axis  (Default: `Repeat`)
- **WrapModeW**: The `TextureWrapMode` along the W (depth, only relevant for Texture3D) axis. (Default: `Repeat`)
- **AnisoLevel**: The `anisoLevel` of the texture. A value of 0 disables filtering, 16 equals full filtering. (Default: `9`)
  - VRChat uses forced anisotropic filtering. When the anisoLevel value is between 1 and 9, Unity sets the anisoLevel to 9. If the value is higher than 9, Unity clamps it between 9 and 16.
- **MaterialProperty**: Overrides which `MaterialProperty` to apply the downloaded texture to, if a `material` was specified in `DownloadImage`. (Default: `_MainTex`)

### IVRCImageDownload

Contains information about the downloaded image. Returned by `VRCImageDownloader`'s `DownloadImage` function, by `OnImageLoadSuccess`, and by `OnImageLoadError`.  
Note that many of these fields will be invalid until the download has completed or failed.

- **Get Error**: Gets the `VRCImageDownloadError` associated with the event. 
- **Get Errormessage**: Gets the error message as a `string`.  
- **Get Material**: Gets the Material sent into the `DownloadImage` function.  
- **Get Progress**:`Gets the progress of the image download as a`float\` between 0 and 1. Use this to track the progress of the download, i. e. for custom loading bars.
- **Get Result**: The `Texture2d` of the downloaded image.  
- **Get SizeInMemoryBytes**: Gets the size of the texture in bytes as an `int`. 
- **Get State**: Gets the `VRCImageDownloadState` indicating the state of the image download.  
- **Get TextureInfo**: The texture info given to the DownloadImage function (TextureInfo).  
- **Get Udonbehavior**: Gets the given udonbehavior the events of the download image are being sent to (UdonBehavior).
- **Get URL**: Gets the `VRCURL` of the image download.

VRChat automatically selects the <UnityVersionedLink versionKey="minor" url="https://docs.unity3d.com/<VERSION>/Documentation/ScriptReference/TextureFormat.html">texture format</UnityVersionedLink> of the downloaded image.
- Images with an alpha channel as loaded as RGBA32, RGB64, etc.
- Images without an alpha channel are loaded as RGB24, RGB48, etc.
- Greyscale images are loaded as R8, R16, etc.

#### Dispose

Cleans up the `IVRCImageDownload`. Unloads the downloaded texture and frees up the memory it was using.

Unlike the dispose method of `VRCImageDownloader`, this will only dispose this individual download and its associated texture, leaving other downloads and their textures intact.

Disposing an `IVRCImageDownload` will change its `State` to `Unloaded`.

### VRCImageDownloadState

Indicates the state of the image download in `IVRCImageDownload`:

- **Pending**: Not been started or still in progress.
- **Error**: Download failed an error (see `VRCImageDownloadError`).
- **Complete**: Download complete, texture is ready to use.
- **Unloaded**: Pending garbage collection after `Dispose` has been called on `IVRCImageDownload`.
- **Unknown**: Unknown state.

### VRCImageDownloadError

When an image download fails, `OnImageLoadError` is called. `IVRCImageDownload`'s `Error` field will contain one of the following error states:

- **InvalidURL**: The download URL used in `DownloadImage` is invalid.
- **AccessDenied**: Access to the URL was denied.
- **InvalidImage**: The downloaded image is invalid.
- **DownloadError**: A web request error occured.
- **Unknown**: Unknown error state.

## Events

* **OnImageLoadSuccess**: Returns `IVRCImageDownload`. Called when a `VRCImageDownloader` has successfully download an image.
* **OnImageLoadError**: Returns `IVRCImageDownload`. Called when a `VRCImageDownloader` has failed to download an image.
