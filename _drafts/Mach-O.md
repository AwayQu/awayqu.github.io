


# Load Commands

## LC_RPATH


* @executable_path 
    这个变量表示可执行程序所在的目录. 比如 

* @loader_path 
  这个变量表示每一个被加载的 binary (包括App, dylib, framework,plugin等) 所在的目录.

  `/path/FlashPlayer.plugin/Contents/MacOS/FlashPlayer`
  依赖
  `/path/FlashPlayer.plugin/Contents/Frameworks/XPSSO.dylib`


  把 `XPSSO.dylib` 的 `INSTALL_PATH` 设置为`@loader_path/../Frameworks/`
  加载`FlashPlayer`这个plugin时, loader_path == `/path/FlashPlayer.plugin/Contents/MacOS/`, 相对路径保证改变整个plugin路径,一直可以正确加载 依赖 `XPSSO.dylib`


* @rpath 

和前面两个不同, 它只是一个保存着一个或多个路径的变量. 比如 XPSSO.dylib 被两个 .app 使用, 且被包含的路径不同。比如：

softA.app/Contents/MacOS/dylib/XPSSO.dylib
softB.app/Contents/MacOS/Frameworks/XPSSO.dylib

将 XPSSO.dylib 的 INSTALL_PATH 设置成 @loader_path/../dylib 或 @loader_path/../Frameworks 都只能满足其中一个 .app 的需求. 要解决这个问题, 就可以用 @rpath. 将 XPSSO.dylib 的 INSTALL_PATH 设置成 @rpath, 然后在编译 softA.app, softB.app 时分别指定 @rpath 为 @loader_path/../dylib, @loader_path/../Frameworks, 问题得到了解决. @rpath 的另一个优点是可以设置多个路径. 如果 softA.app 还需要使用另一个 .plugin (假设它的 INSTALL_PATH 也设置成了 @rpath), 位于 @loader_path/../plugin, 把这个路径加到 @rpath 即可.




http://www.tanhao.me/pieces/1361.html/

http://matthew-brett.github.io/docosx/mac_runtime_link.html

https://www.cnblogs.com/csuftzzk/p/mac_run_path.html