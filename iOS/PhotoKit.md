# PhotoKit

`PhotoKit` 是 iOS 8 引入的库

PhotoKit 中的所有 PhotoKit 对象都继承 `PHObject` 抽象类。

- `PHAsset`: 代表用户照片库中一个单独的资源
- `PHFetchOptions`: 获取资源时的参数，可以传 nil，即使用系统默认值
- `PHAssetCollection`: 代表一个资源集合。一个 `PHAssetCollection` 代表照片库中的一个相册。`PHAssetCollection` 是 `PHCollection` 的子类。
- `PHCollectionList`: 表示一组 PHCollections。可以嵌套 `PHCollectionList` 和自身类型，还支持多重嵌套。在系统的照片应用中：照片---时刻---精选---年度，就是一个例子
- `PHFetchResult`: 表示一系列的资源结果集合，也可以是相册的集合，从 `PHCollection` 的类方法中获得
- `PHImageManager`: 用于处理资源的加载，加载图片的过程带有缓存处理，可以通过传入一个 `PHImageRequestOptions` 控制资源的输出尺寸等规格
- `PHImageRequestOptions`: 控制加载图片时的一系列参数

`PHAsset`、`PHAssetCollection` 和 `PHCollectionList` 都是轻量级的不可变对象，使用这些类时并没有将其代表的图片、视频或者集合载入内存中，要使用其代表的图片或者视频，需要通过 `PHImageManager` 类来请求。

## 权限获取
```Swift
let currentStatus = PHPhotoLibrary.authorizationStatus()
switch currentStatus {
        case .notDetermined: 
            //用户没有进行授权，发起授权许可
            PHPhotoLibrary.requestAuthorization({ (status) in
                if status == .authorized {
                    //继续
                }
            })
        case .authorized:
            //继续
        case .denied, .restricted:
            //用户拒绝授权，或者相机异常

        }
```

## 获取相册

获取相册是由`PHAssetCollection` 或 `PHCollectionList` 使用类方法获取的。

```Swift

	//options 参数给了我们一个对结果进行过滤和排序的途径。  
   //这里按照估计的每个相册的照片数排序
     let options = PHFetchOptions()
     let descriptor = NSSortDescriptor(key: "estimatedAssetCount", ascending: false)
   options.sortDescriptors = [descriptor]

	//获取系统相册的方法:  
     let systemAlbums = PHAssetCollection.fetchAssetCollections(with: .smartAlbum, subtype: .albumRegular, options: options)
        
	//获取用户创建的相册的方法:  
     let userAlbums = PHAssetCollection.fetchAssetCollections(with: .album, subtype: .albumRegular, options: options)
     let userAlbums = PHAssetCollection.fetchTopLevelUserCollections(with: nil)
	
	//获取所有资源的集合
  	  let assetsFetchResults  = PHAsset.fetchAssetsWithOptions(nil)
	
	
```

两种方法的返回值都是 `PHFetchResult`，可以用类似 NSArray 的接口来访问结果内的集合

```Swift

enum PHAssetCollectionType : Int {
    case Album //从 iTunes 同步来的相册，以及用户在 Photos 中自己建立的相册
    case SmartAlbum //经由相机得来的相册
    case Moment //Photos 为我们自动生成的时间分组的相册
}

enum PHAssetCollectionSubtype : Int {
    case AlbumRegular //用户在 Photos 中创建的相册，也就是所谓的逻辑相册
    case AlbumSyncedEvent //使用 iTunes 从 Photos 照片库或者 iPhoto 照片库同步过来的事件。然而，在iTunes 12 以及iOS 9.0 beta4上，选用该类型没法获取同步的事件相册，而必须使用AlbumSyncedAlbum。
    case AlbumSyncedFaces //使用 iTunes 从 Photos 照片库或者 iPhoto 照片库同步的人物相册。
    case AlbumSyncedAlbum //做了 AlbumSyncedEvent 应该做的事
    case AlbumImported //从相机或是外部存储导入的相册。
    case AlbumMyPhotoStream //用户的 iCloud 照片流
    case AlbumCloudShared //用户使用 iCloud 共享的相册
    case SmartAlbumGeneric //文档解释为非特殊类型的相册，主要包括从 iPhoto 同步过来的相册。
    case SmartAlbumPanoramas //相机拍摄的全景照片
    case SmartAlbumVideos //相机拍摄的视频
    case SmartAlbumFavorites //收藏文件夹
    case SmartAlbumTimelapses //延时视频文件夹，同时也会出现在视频文件夹中
    case SmartAlbumAllHidden //包含隐藏照片或视频的文件夹
    case SmartAlbumRecentlyAdded //相机近期拍摄的照片或视频
    case SmartAlbumBursts //连拍模式拍摄的照片，在 iPad mini 上按住快门不放就可以了，但是照片依然没有存放在这个文件夹下。
    case SmartAlbumSlomoVideos //Slomo 是 slow motion 的缩写，高速摄影慢动作解析，在该模式下，iOS 设备以120帧拍摄。
    case SmartAlbumUserLibrary //所有相机拍摄的照片或视频都会出现在该相册中，而且使用其他应用保存的照片也会出现在这里。
    case Any //包含所有类型
}

```

## 获取图片的元数据

上面获取的是相册的列表，现在从列表中取出每个图片的元数据，利用 `PHAsset` 的类方法获取图片元数据

```Swift

	  //这里根据照片的创建时间，对相册内的照片进行排序

	 let options = PHFetchOptions()
     let descriptor = NSSortDescriptor(key: "creationDate", ascending: true)
     options.sortDescriptors = [descriptor]

     // fetchResult 是相册中所有图片的元数据，collection 是前面获取相册列表中的对象

     let fetchResult = PHAsset.fetchAssets(in: collection, options: options)


```

## 图片元数据

#### Asset 类型
通过 `mediaType` 可以知道这个 asset 类型

- image
- video
- audio

#### HDR 和全景照片
可以通过照片资源的 `mediaSubtypes` 验证是否启用 HDR 和 全景模式

- photoPanorama: 全景模式
- photoHDR: HDR

#### 其他属性

- isHidden：是否隐藏
- isFavorite：是否收藏
- creationDate：创建的时间
- modificationDate：最后一次修改的时间
- location：位置

#### 连拍照片
如果 `PHAsset` 的 `representsBurst` 属性为 true，表示这个资源是一系列连拍照片中的代表照片。它还有一个属性是 `burstIdentifier`，如果想要获取连拍照片中的其他照片，可以将 `burstIdentifier` 传入 `fetchAssets(withBurstIdentifier burstIdentifier: String, options: PHFetchOptions?)` 方法来获取。

## 照片加载
Photokit 提供了一个类： `PHImageManager`。

#### 请求图像
图片的请求是通过 `requestImageForAsset` 方法进行。

```Swift
	PHImageManager.default().requestImage(for:asset,
	 targetSize: size, contentMode: .aspectFill,
	  options: options,
	  resultHandler: { (image, info) in
	  	//得到 image 后的处理
	  })
```

 这个方法接受的参数:

- `PHAsset`。
- `CGSize`，希望获取的图片大小。
- `contentMode`
- 和图片的其他可选项（通过 `PHImageRequestOptions` 参数对象设置）
- 获得 UIImage 后的回调

这个方法有一个 `PHImageRequestID` 的返回值，可以用来取消这个请求。  
获取视频的缩略图也是用这个方法，传入的 `PHAsset` 是视频的资源对象即可。

#### 图像的尺寸剪裁

`targetSize` 和 `contentMode` 这两个参数决定了图片是按比例缩放，还是按比例填充的方式放到目标大小内。  
需要返回原图时这两个参数是：`PHImageManagerMaximumSize` 和 `PHImageContentMode.Default`。

#### PHImageRequestOptions
`PHImageRequestOptions` 的 `deliveryMode` 可以设置三种策略:

- Opportunistic：先传递较低质量的版本，随后传递高质量的，可能调用多次回调。
- HighQualityFormat：高质量的图片，较长的加载时间
- FastFormat：更快的加载速度，且牺牲一点图片质量

`isNetworkAccessAllowed` 是否允许从 iCloud 下载图片，默认是 `NO`

`isSynchronous`  设置是否同步。当设为 `true` 时， `deliveryMode` 属性就会被忽略，并被当做 `. HighQualityFormat` 处理。 `isSynchronous` 默认为 `NO`。

`progressHandler` 下载进度的一个 block，当从 iCloud 下载照片时，它会被图像管理器自动调用。

`version` 
PhotoKit 允许应用对照片进行无损的修改。对编辑过的照片，系统会对单独保存一份原始照片的拷贝和针对应用的调整数据。当用图像管理器获取资源时，你可以指定哪个版本的图像资源应该通过 `resultHandler` 被递送。这可以通过设置 `version` 属性来做到：`.Current` 会递送包含所有调整和修改的图像；`.Unadjusted` 会递送未被施加任何修改的图像；`.Original` 会递送原始的、最高质量的格式的图像 (例如 RAW 格式的数据。而当将属性设置为 `.Unadjusted` 时，会递送一个 JPEG)。

#### 结果回调
回调是一个block，包含一个 `UIImage` 变量和一个 `info` 字典。根据请求的参数，block 有可能被多次调用。  
`info` 字典提供了关于当前请求状态的信息，比如:

- 图像是否必须从 iCloud 请求 (如果你初始化时将 networkAccessAllowed 设置成 false，那么就必须重新请求图像) —— PHImageResultIsInCloudKey 。
- 当前递送的 UIImage 是否是最终结果的低质量格式。当高质量图像正在下载时，这个可以让你给用户先展示一个预览图像 —— PHImageResultIsDegradedKey。
- 请求 ID，以及请求是否已经被取消 —— PHImageResultRequestIDKey 和 PHImageCancelledKey。
- 如果没有图像提供给 result handler，字典内还会有一个错误信息 —— PHImageErrorKey。

#### 缓存
如果要加载大量的资源图片的缩略图时，预先将一些图片加载到内存有时是非常有用的。  
PhotoKit 提供了一个 `PHImageManager` 的子类来处理这种特定的使用场景 -- `PHImageCachingManager`。

`PHImageCachingManager` 提供了一个关键方法 —— `startCachingImagesForAssets(...)` 来缓存图像。你传入一个 `PHAssets` 类型的数组，一些请求参数，以及一些请求单个图像时即将用到的可选项。此外，还有一些方法可以让你通知缓存管理器来停止缓存特定资源列表，以及停止缓存所有图像。

获取图片用的是父类的方法 `requestImageForAsset(...)`，传入的参数必须和 `startCachingImagesForAssets(...)` 传入的参数相同，才能取得之前缓存的图片。  

`PHImageCachingManager` 有一个属性 `allowsCachingHighQualityImages`，它可以让你指定图像管理器是否应该准备高质量图片，默认值为 true。如果滑动效果较差时，可以设置为 false 提高滑动效果。

调用 `requestImageForAsset(...)` 取得图片时如果没有缓存，会自动将该图片进行缓存。

## 导出资源
#### 视频导出
```Swift
      let requestOptions = PHVideoRequestOptions()
        requestOptions.deliveryMode = .highQualityFormat
        requestOptions.isNetworkAccessAllowed = true
        requestOptions.progressHandler = {(progress, error, stop, info) in
          LLog(progress)
        }

      let id = PHImageManager.default().requestExportSession(forVideo: asset,
      options: requestOptions,
        exportPreset: AVAssetExportPresetPassthrough) { (exportSession, info) in

          if let session = exportSession {
              session.outputFileType = AVFileTypeMPEG4
              session.outputURL = filePath

              session.exportAsynchronously {
                  //在这里做导出完成后的处理
              }
          }
      }
```

#### 图片导出
```Swift

      let requestOptions = PHImageRequestOptions()
      requestOptions.isSynchronous = false
      requestOptions.deliveryMode = .highQualityFormat
      requestOptions.isNetworkAccessAllowed = true

      let id = PHImageManager.default().requestImageData(for: asset,
       options: requestOptions,
       resultHandler: { (imageData, uti, imageOrientation, info) in
          //处理 imageData
      })

```

如果需要导出大量的原图，使用 `requestImageForAsset` 会大量增加应用的内存占用，会导致应用崩溃，使用 `requestImageData` 的话，内存占用会大大降低，所以这里需要使用 `requestImageData`。
