## Live Photos

WWDC 15 没有发表，2015 年 9 月的 Special Event 上随 iPhone 6s 一同发表。

iPhone 6/6s 可以拍摄，内：3 秒时间的视频和一张 JPG 组成，iOS 9 以后的设备都可以播放。

iOS 9.1 中 live photos 有关的 API 已经公开：

* 显示，再生：`PHLivePhotoView`
* 分享：`PHAssetResource`

对 Live Photo 做编辑的时候，需要注意对所有的 frame 都作出调整。

* Vine 支持 6 秒短视频，相比于 Vine，Live Photo 更加内敛。现时点，Facebook 和 Instagram 暂时都还没有对应 Live Photo。

向 Simulator 中传送 Live Photo 的方式：

1. iPhone 拍摄
2. iPhone -> 传输到照片 app
3. 使用 iCloud 导入，不能直接 drag & drop (麻烦)。

也许可以考虑写个工具来在实机和 Simulator 之间进行转换。

## Try Swift

* Day 1
    * Syo Ikeda - Dive into Swift Ecosystem
    * JP Simard - Practical Cross-Platform Swift
    * Laura Savino - Learning to Read Again
    * Yuta Koshizawa - Three Stories about Error Handling in Swift
    * Boris Bugling - Apple TV
    * Gwendolyn Weston - Keep Calm and Type Erase On
    * Michele Titolo - Protocols and the Promised Land
    * Daniel Steinberg - Blending Culture
    * Tim Oliver - Advanced Graphics with Core Animation
    * Stephanie Shupe - Code for the smart home
    * Cate Huston - How To Be Invisible
* Day 2
    * Ayaka Nonaka - Boundaries in Practice
    * Adam Bell - Prototyping Magic
    * Matthew Gillingham - Protocol Extension: A History
    * Himi Sato: Building Women Who Code in Tokyo
    * Rachel Bobbins - The Design of Everyday Swift
    * Daniel Eggert - Modern Core Data
    * Novall Khan - Swift compiler integration in LLDB
    * Jeff Hui - Creating a Library
    * Yosuke Ishikawa - Protocol-Oriented Programming in Networking
    * Maxim Cramer - Live Design
    * Chris Eidhof - Table View Controllers in Swift
* Day 3
    * Hiroki Kato - Motivation based library abstraction
    * Caesar Wirth - Soaring Swiftly - Server Side Swift
    * Veronica Ray - Real World Mocking in Swift
    * Hector Matos - Hipster Swift
    * Simon Gladman - Advanced Image processing with Core Image
    * Diana Zmuda - How to Train Your Swift: Examples of Computational Statistics in Swift
    * Diniel Haight - xcodeless - the build system
    * Helen Holmes - 10 Ways to Get Designers In Your Swift Codebase
    * Yasuhiro Inami - Parser Combinator in Swift
    * Jesse Squires - Contributing to open source Swift
    * Ash Furrow - An Artsy Testing Tour
