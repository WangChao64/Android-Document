# Android 文件存储
一、内部存储和外部存储

 这里的存储指的是永久非易失的存储（rom），不是内存（ram）。

这里的外部存储不是指 特定的可移动的存储介质（如SD卡），现在很多Android手机都是一体机，普通消费者无法拆卸的，也没有扩展SD卡的的卡槽了，我们把手机本身自带的rom叫机身存储或内置存储。现在的Android设备都将机身存储 划分了内部存储和外部存储，体现内部和外部的区别。如果可扩展可移动存储(如SD卡)，这也是外部存储，这样手机则包含多个外部存储。

注：下面的讨论都是基于Android 4.4之后的,因为在老的Android版本(4.4之前)上 机身存储没有划分内部存储和外部存储，机身存储即内部存储，SD卡即外部存储。

因为外部存储事可移动的 可移除的，存在明显的差异，它们具体不同的使用场景和功能，下面是大致差异，后面再具体说明。

内部存储

可用： 一直可用
访问：应用保存文件在这里，只有应用本身能访问
卸载：卸载应用时,应用的所有文件都会从内部存储中删除
外部存储

可用：非一直可用，可用挂载外部存储作为USB存储，甚至可用移除外部存储
访问：存储在外部存储的文件，很容易被其他应用访问和修改
卸载：卸载应用时，应用在外部存储创建的文件不一定都会被删除掉，只是在getExternalFilesDir()路径下的文件会被删除（后面会具体说明）
 

1、内部存储

 当存储在内部存储时，可用通过下面方法获取合适的File对象。

Context   getFilesDir()： 返回应用程序的内部files目录的File对象。如：/data/user/0/com.flx.testfilestorage/files

Context   getCacheDir()：返回一个应用程序内部临时文件目录的File对象，这个临时文件在不需要时会被删除，在系统可用存储很低时也会被系统删除。如：/data/user/0/com.flx.testfilestorage/cache

 

应用files目录中文件的操作

创建：这里给出了如下两种方式

构造File对象，包含了目录和文件名。通过createNewFile()进行创建，如果文件不存在则创建。
File createFiles = new File(context.getFilesDir(), "testfile.txt");
try {
    createFiles.createNewFile();
} catch (IOException e) {
    Log.d( TAG, "files err:"+e.getMessage() );
}
通过Context的api直接操作，通过openFileOutput直接获取FileOutputStream输出流对象，如果文件不存在，则直接创建。
注：openFileOutput的第二个参数（mode）文件的操作模式，其他值在API 17以后被弃用，只能使用MODE_PRIVATE。在SharedPreferences中有详细说明：https://www.cnblogs.com/fanglongxiang/p/11390013.html

复制代码
        try {
            //mode参数注意下,这里使用的Context.MODE_PRIVATE
            FileOutputStream outputStream = context.openFileOutput( "testfile22.txt", Context.MODE_PRIVATE );
            outputStream.write( "Use OutputStream Create file\n".getBytes() );
            outputStream.close();
        } catch (IOException e) {
            Log.d( TAG, "outputStream err:"+e.getMessage() );
        }
复制代码
 

获取文件：

同上，通过目录和文件名 构造File对象
1
File createFiles = new File(context.getFilesDir(), "testfile.txt");
上面通过openFileOutput获取文件输出流 可写文件（不存在则创建），也可通过openFileInput获取文件输入流 进行读取文件内容。
FileOutputStream outputStream = context.openFileOutput( "testfile22.txt", Context.MODE_PRIVATE );//输出流
FileInputStream fileInputStream = context.openFileInput( "testfile22.txt" );//输入流
还可直接获取files里的所有文件列表
1
String[] arrFiles = context.fileList();
　　

读写文件：

直接构造文件输出输入流读写文件。
FileOutputStream fileOutputStream = context.openFileOutput( "testfile22.txt", Context.MODE_PRIVATE );
FileOutputStream fileOutputStream2 = new FileOutputStream( createFiles );
 
FileInputStream fileInputStream = context.openFileInput( "testfile22.txt" );
FileInputStream fileInputStream2 = new FileInputStream( createFiles );
　　

删除文件：

File对象直接调用delete()方法删除
createFiles.delete();
Context直接通过文件名删除
context.deleteFile( "testfile22.txt" );
　　

创建临时文件:

上述files目录操作方式同样适用（File对象需要指定路径的可以，而Context直接通过文件名的则不可以），另外添加一个createTempFile()的方法。

File.createTempFile( "tempfile", null, context.getCacheDir() );
　　

2、外部存储

首先，简单介绍下两个方法的差异以及主外部存储。

先看下这段代码，

复制代码
String state = Environment.getExternalStorageState();
File externalFile = context.getExternalFilesDir( null );
File[] externalFiles = context.getExternalFilesDirs( Environment.DIRECTORY_PICTURES );
for (File file : externalFiles) {
    Log.d( TAG, "state="+ state + ";\nexternalFiles=" + file + ";\nexternalFile="+externalFile);
    try {
        FileOutputStream fileOutputStream = new FileOutputStream( new File( file, "aaaa.txt" ) );
        fileOutputStream.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
复制代码
运行的手机支持SD卡 并插入了一张SD卡，看看运行结果

2019-07-15 14:46:07.819 12704-12704/com.flx.testfilestorage D/flx_storage: state=mounted;
    externalFiles=/storage/emulated/0/Android/data/com.flx.testfilestorage/files/Pictures;
    externalFile=/storage/emulated/0/Android/data/com.flx.testfilestorage/files
2019-07-15 14:46:07.821 12704-12704/com.flx.testfilestorage D/flx_storage: state=mounted;
    externalFiles=/storage/553C-0E05/Android/data/com.flx.testfilestorage/files/Pictures;
    externalFile=/storage/emulated/0/Android/data/com.flx.testfilestorage/files
从上面看到，getExternalFilesDirs获取的有两个外部存储，getExternalFilesDir是一个。这两个外部存储，一个是主外部存储 即手机本身存储中分为 内部存储和外部存储的 外部存储部分，另一个是SD卡的挂载路径。

getExternalFilesDir()，获取就是主外部存储路径。

getExternalFilesDirs()，获取所有外部存储的路径，包括本身的外部存储 和 扩展出来的存储(如SD卡)。

在一开始就说过，应用存储到外部存储的文件 当应用卸载时只有getExternalFilesDir()路径下的会被删除。

上面代码在log后，所有外部存储路径下 都创建了aaaa.txt的文件，实际操作结果也是符合的，当卸载应用时，/storage/553C-0E05/Android/data/com.flx.testfilestorage/files/这个下面的aaaa.txt 仍然存在的。

 


可用性判断： 

由于外部存储不一定一直有用，可能被移除等情况，所以使用外部存储需要进行判断，外部存储是否可用。

Environment.getExternalStorageState([File path])这个方法可以返回外部存储的状态,不过它只返回主外部存储的状态。从源码中，也能直接的看出来

 

 保存文件到公共目录：

保存文件到外部存储的公共目录，其他应用也能直接访问，有以下两种方法（非直接创建读写文件）

媒体文件如图片、音频文件、录像文件等，使用MediaStore的API进行操作。
保存其他文档文件，如PDF，使用ACTION_CREATE_DOCUMENT intent操作。这个就属于SAF(Storage Access Framework)相关的了。
注：如果媒体文件，不希望被 Media Scanner 扫描，可以在相应目录创建一个空的文件命名为 .nomedia  。这样就能阻止Media Scanner的扫描和读取文件 并 通过MediaStore的API提供给其他应用使用。

 

保存文件到私有目录：

应用保存文件到外部存储的私有目录，可以使用getExternalFilesDir()方法。具体实例 参考外部存储开始的示例。

注意：1.注意getExternalFilesDir()和getExternalFilesDirs()的区别、使用。

　　    2.getExternalFilesDir()里的文件 会随着应用的卸载而删除。

　　　3.注意这两个方法的参数，尽量正确使用API提供的常量，这样系统才能正确处理文件。如：使用Environment.DIRECTORY_RINGTONES，系统media scanner能够识别到是铃声而不是音乐。

 

二、存储路径

从上面的介绍可用看到，应用存储在 内部存储和外部存储上的私有目录（getExternalFilesDir()）下的，是和应用相关的，在应用卸载时 对应目录文件会被删除。

下面列举了常用的存储路径，

复制代码
        int i = 0;
        for (File file:context.getExternalFilesDirs( null )) {
            Log.d( TAG, " \ncontext.getExternalFilesDirs()[" + ++i + "]=" + file );
        }
        i=0;
        Log.d( TAG, " \ncontext.getExternalFilesDir()=" + context.getExternalFilesDir( null ) );
        Log.d( TAG, " \ncontext.getCacheDir()=" + context.getCacheDir() );
        Log.d( TAG, " \ncontext.getCodeCacheDir()=" +  context.getCodeCacheDir());
        Log.d( TAG, " \ncontext.getDatabasePath()=" + context.getDatabasePath( "external.db" ));
        Log.d( TAG, " \ncontext.getDataDir()=" + context.getDataDir() );
        Log.d( TAG, " \ncontext.getDir()=" + context.getDir( null, Context.MODE_PRIVATE ) );
        Log.d( TAG, " \ncontext.getExternalCacheDir()=" + context.getExternalCacheDir() );
        Log.d( TAG, " \ncontext.getFilesDir()=" + context.getFilesDir() );
        Log.d( TAG, " \ncontext.getObbDir()=" + context.getObbDir() );
        for (File file:context.getExternalCacheDirs()) {
            Log.d( TAG, " \ncontext.getExternalCacheDirs()[" + ++i + "]=" + file );
        }

        Log.d( TAG, " \nEnvironment.getDataDirectory()=" + Environment.getDataDirectory() );
        Log.d( TAG, " \nEnvironment.getExternalStorageDirectory()="+Environment.getExternalStorageDirectory() );
        Log.d( TAG, " \nEnvironment.getDownloadCacheDirectory()="+Environment.getDownloadCacheDirectory() );
        Log.d( TAG, " \nEnvironment.getRootDirectory()="+Environment.getRootDirectory() );
        Log.d( TAG, " \nEnvironment.getExternalStoragePublicDirectory()= "+Environment.getExternalStoragePublicDirectory( Environment.DIRECTORY_PICTURES ) );
        
复制代码
 

log打印出具体路径：
2019-09-11 16:38:08.799 19469-19469/com.flx.testfilestorage D/flx_storage: 
    context.getExternalFilesDirs()[1]=/storage/emulated/0/Android/data/com.flx.testfilestorage/files
2019-09-11 16:38:08.799 19469-19469/com.flx.testfilestorage D/flx_storage: 
    context.getExternalFilesDirs()[2]=/storage/553C-0E05/Android/data/com.flx.testfilestorage/files
2019-09-11 16:38:08.803 19469-19469/com.flx.testfilestorage D/flx_storage: 
    context.getExternalFilesDir()=/storage/emulated/0/Android/data/com.flx.testfilestorage/files
2019-09-11 16:38:08.805 19469-19469/com.flx.testfilestorage D/flx_storage: 
    context.getCacheDir()=/data/user/0/com.flx.testfilestorage/cache
2019-09-11 16:38:08.806 19469-19469/com.flx.testfilestorage D/flx_storage: 
    context.getCodeCacheDir()=/data/user/0/com.flx.testfilestorage/code_cache
2019-09-11 16:38:08.807 19469-19469/com.flx.testfilestorage D/flx_storage: 
    context.getDatabasePath()=/data/user/0/com.flx.testfilestorage/databases/external.db
2019-09-11 16:38:08.807 19469-19469/com.flx.testfilestorage D/flx_storage: 
    context.getDataDir()=/data/user/0/com.flx.testfilestorage
2019-09-11 16:38:08.808 19469-19469/com.flx.testfilestorage D/flx_storage: 
    context.getDir()=/data/user/0/com.flx.testfilestorage/app_null
2019-09-11 16:38:08.813 19469-19469/com.flx.testfilestorage D/flx_storage: 
    context.getExternalCacheDir()=/storage/emulated/0/Android/data/com.flx.testfilestorage/cache
2019-09-11 16:38:08.814 19469-19469/com.flx.testfilestorage D/flx_storage: 
    context.getFilesDir()=/data/user/0/com.flx.testfilestorage/files
2019-09-11 16:38:08.818 19469-19469/com.flx.testfilestorage D/flx_storage: 
    context.getObbDir()=/storage/emulated/0/Android/obb/com.flx.testfilestorage
2019-09-11 16:38:08.823 19469-19469/com.flx.testfilestorage D/flx_storage: 
    context.getExternalCacheDirs()[1]=/storage/emulated/0/Android/data/com.flx.testfilestorage/cache
2019-09-11 16:38:08.824 19469-19469/com.flx.testfilestorage D/flx_storage: 
    context.getExternalCacheDirs()[2]=/storage/553C-0E05/Android/data/com.flx.testfilestorage/cache
2019-09-11 16:38:08.824 19469-19469/com.flx.testfilestorage D/flx_storage: 
    Environment.getDataDirectory()=/data
2019-09-11 16:38:08.828 19469-19469/com.flx.testfilestorage D/flx_storage: 
    Environment.getExternalStorageDirectory()=/storage/emulated/0
2019-09-11 16:38:08.828 19469-19469/com.flx.testfilestorage D/flx_storage: 
    Environment.getDownloadCacheDirectory()=/data/cache
2019-09-11 16:38:08.828 19469-19469/com.flx.testfilestorage D/flx_storage: 
    Environment.getRootDirectory()=/system
2019-09-11 16:38:08.832 19469-19469/com.flx.testfilestorage D/flx_storage: 
    Environment.getExternalStoragePublicDirectory()= /storage/emulated/0/Pictures
Context获取的都是包含包名的路径。仔细看下就能知道 方法获取的是手机中的路径。

这些方法获取到的都是File对象，可用使用getTotalSpace()或getFreeSpace()获取总空间大小或剩余空间大小，注意单位时byte.

 

下面是 /data/user/0/com.flx.testfilestorage/ 在手机中的显示，不是所有应用下面的文件夹都有的，一些是上面运行产生的。正常如果没有对应文件 则对应文件夹是不存在的，如没有数据库文件 则databases是没有的，如果有SharedPreferences数据则还会有shared_prefs文件夹。不过cache,code_cache,files一般都是有的。



 

 路径疑问：

以前随笔中介绍的，很多路径是/data/data/xxx，如SharedPreferences存储在/data/data/<packagename>/shared_prefs/下，数据库存储在/data/data/<package_name>/databases下。而上面获取到的是/data/user/0/xxxx下，这是怎么回事呢？

通过Android Studio看下/data/data/com.flx.testfilestorage/ 里面的内容可以看到是一样的.



其实,/data/user/0/ 就是链接的/data/data/.所以他们的目标目录都是/data/data/xxx下的。



 

 还可以通过命令查看，第一位的l就是表示一个链接。

