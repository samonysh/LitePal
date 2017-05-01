# LitePal for Android 

Samon Yu

参考：

1. https://github.com/LitePalFramework/LitePal
2. http://blog.csdn.net/xinpengfei521/article/details/57413395
3. http://www.jianshu.com/p/557682e0a9f0

## 特性

1. 使用对象-关系模型（ORM）
2. 0-认证
3. 自动表处理（例如：创造、修改、删除表）
4. 支持大量的数据库
5. 避免SQL语句的API



## 最新下载

1.5.1



## 入门（配置）

### 1.加入工程

使用AS

首先记得导入下载的文件至lib中，并添加至lib

编辑 **build.gradle** 文件添加以下代码：

```java
dependencies {
    compile 'org.litepal.android:core:1.5.1'
}
```

### 2.配置 litepal.xml 

新建一个文件加入你的工程中的assets文件夹中（要新建，与res并列），文件名 litepal.xml 。这仅仅只是配置文件, 这个配置也是非常简单. 

* **dbname** 配置你的工程的数据库名称. 
* **version** 配置数据库版本号. 每次你想升级数据库, 就将这个值+1. 
* **list** 配置映射的JavaBean类. 
* **storage** 配置你的数据库将会保存在哪里. **internal** 和 **external** 只有这两种合法的选择.

```xml
<?xml version="1.0" encoding="utf-8"?>
<litepal>
    <!--
    	Define the database name of your application. 
    	By default each database name should be end with .db. 
    	If you didn't name your database end with .db, 
    	LitePal would plus the suffix automatically for you.
    	For example:    
    	<dbname value="demo" />
    -->
    <dbname value="demo" />

    <!--
    	Define the version of your database. Each time you want 
    	to upgrade your database, the version tag would helps.
    	Modify the models you defined in the mapping tag, and just 
    	make the version value plus one, the upgrade of database
    	will be processed automatically without concern.
			For example:    
    	<version value="1" />
    -->
    <version value="1" />

    <!--
    	Define your models in the list with mapping tag, LitePal will
    	create tables for each mapping class. The supported fields
    	defined in models will be mapped into columns.
    	For example:    
    	<list>
    		<mapping class="com.test.model.Reader" />
    		<mapping class="com.test.model.Magazine" />
    	</list>
    -->
    <list>
    </list>
    
    <!--
        Define where the .db file should be. "internal" means the .db file
        will be stored in the database folder of internal storage which no
        one can access. "external" means the .db file will be stored in the
        path to the directory on the primary external storage device where
        the application can place persistent files it owns which everyone
        can access. "internal" will act as default.
        For example:
        <storage value="external" />
    -->
    
</litepal>
```

### 3.配置 LitePalApplication

在 AndroidManifest.xml 配置如下：

```xml
<manifest>
    <application
        android:name="org.litepal.LitePalApplication"
        ...
    >
        ...
    </application>
</manifest>
```

也可以在自己的应用里设置：

```xml
<manifest>
    <application
        android:name="com.example.MyOwnApplication"
        ...
    >
        ...
    </application>
</manifest>
```



然后在自己的应用里调用 **LitePal.initialize(context)** ：

```java
public class MyOwnApplication extends AnotherApplication {

    @Override
    public void onCreate() {
        super.onCreate();
        LitePal.initialize(this);
    }
    ...
}
```

> 调用越早越好，例如在 **onCreate()** 方法中。并且总是记得去使用application context参数. 不要去使用activity 或者 service的实例作为参数, 否则会导致内存泄漏问题。*待理解*



## 入门（使用）

### 1. 创建表

首先定义模型。例子：

album 与song

```java
public class Album extends DataSupport {
	
    @Column(unique = true, defaultValue = "unknown")
    private String name;
	
    private float price;
	
    private byte[] cover;
	
    private List<Song> songs = new ArrayList<Song>();

    // generated getters and setters.
    ...
}
```

```java
public class Song extends DataSupport {
	
    @Column(nullable = false)
    private String name;
	
    private int duration;
	
    @Column(ignore = true)
    private String uselessField;
	
    private Album album;

    // generated getters and setters.
    ...
}
```

> @Column()的作用是对属性进行限制
>
> 记得生成 get & set 函数

将定义的模型标记在litepal.xml中，例：

```xml
<list>
    <mapping class="com.android.app.src.main.java.Album" />
    <mapping class="com.android.app.src.main.java.Song" />
</list>
```

这样就自动生成了两个表。

### 2.升级表

直接在新建模型的类上面更改即可。

```java
public class Album extends DataSupport {
	
    @Column(unique = true, defaultValue = "unknown")
    private String name;
	
    @Column(ignore = true)
    private float price;
	
    private byte[] cover;
	
    private Date releaseDate;
	
    private List<Song> songs = new ArrayList<Song>();

    // generated getters and setters.
    ...
}
```

> 版本号自己改就行

下次你操作数据库的时候这些表就会被升级， 一个 **releasedate** 字段将会被添加到 **album** 表并且原来的**price** 字段将会被移除. 所有数据在 **album** 表中处理那些被移除的字段。

但是有一些升级表的情况LitePal不能处理升级表被清理的情况，例如：

- 添加一个属性 `unique = true`.
- 改变属性 `unique = true`.
- 把属性变为 `nullable = false`.

要注意上面的这些情况将会造成数据的丢失.

### 3.保存数据

新建一个模型的类的实例，然后使用 set 函数构建字段，最后save()。

```java
Album album = new Album();
album.setName("album");
album.setPrice(10.99f);
album.setCover(getCoverImageBytes());
album.save();
Song song1 = new Song();
song1.setName("song1");
song1.setDuration(320);
song1.setAlbum(album);
song1.save();
Song song2 = new Song();
song2.setName("song2");
song2.setDuration(356);
song2.setAlbum(album);
song2.save();
```

### 4.更新数据

两种方法：

一是先找要更新的字段，然后对字段使用set进行更新，最后save：

```java
Album albumToUpdate = DataSupport.find(Album.class, 1);
albumToUpdate.setPrice(20.99f); // raise the price
albumToUpdate.save();
```

二是首先建立模型的类的实例，使用set设置更改的内容，最后用update或updateAll更新：

```java
Album albumToUpdate = new Album();
albumToUpdate.setPrice(20.99f); // raise the price
albumToUpdate.update(id);
```

```java
Album albumToUpdate = new Album();
albumToUpdate.setPrice(20.99f); // raise the price
albumToUpdate.updateAll("name = ?", "album");
```

### 5.删除数据

```java
DataSupport.delete(Song.class, id);
```

```java
DataSupport.deleteAll(Song.class, "duration > ?" , "350");
```

### 6.查询数据

普通版：

Find a single record from song table with specified id:

```java
Song song = DataSupport.find(Song.class, id);
```

全部：

Find all records from song table:

```java
List<Song> allSongs = DataSupport.findAll(Song.class);
```

带要求的查询：

Constructing complex query with fluent query:

```java
List<Song> songs = DataSupport.where("name like ?", "song%").order("duration").find(Song.class);
```



待更新……

