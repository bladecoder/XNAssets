# XNAssets
[![NuGet](https://img.shields.io/nuget/v/XNAssets.svg)](https://www.nuget.org/packages/XNAssets/) [![Build status](https://ci.appveyor.com/api/projects/status/j1q2injprkq3j18p?svg=true)](https://ci.appveyor.com/project/RomanShapiro/xnassets) [![Chat](https://img.shields.io/discord/628186029488340992.svg)](https://discord.gg/ZeHxhCY)

XNAssets is MonoGame/FNA asset management library that - unlike MonoGame Content Pipeline - loads raw assets.

# Adding Reference
There are two ways of referencing XNAssets in the project:
1. Through nuget(works only for MonoGame): https://www.nuget.org/packages/xnassets
2. As repo(works for both MonoGame and FNA):
    
    a. Clone this repo.
    
    b. Execute `git submodule update --init --recursive` within the folder the repo was cloned to.
    
    c. Add src/XNAssets.MonoGame.csproj or src/XNAssets.FNA.csproj to the solution.
    
      * If FNA is used, then the folder structure is expected to be following: ![Folder Structure](/images/FolderStructure.png)
    
# Creating AssetManager
In order to create AssetManager two parameters must be passed to its constructor: GraphicsDevice and [IAssetResolver](https://github.com/rds1983/XNAssets/blob/master/src/XNAssets/Assets/IAssetResolver.cs). Latter is simple interface that opens asset stream by its name.

XNAAssets provides 3 implementation of IAssetResolver:
  * [FileAssetResolver](https://github.com/rds1983/XNAssets/blob/master/src/XNAssets/Assets/FileAssetResolver.cs) that opens Stream using File.OpenRead. Sample AssetManager creation code:
```c#
    FileAssetResolver assetResolver = new FileAssetResolver(Path.Combine(PathUtils.ExecutingAssemblyDirectory, "Assets"));
    AssetManager assetManager = new AssetManager(GraphicsDevice, assetResolver);
```

  * [ResourceAssetResolver](https://github.com/rds1983/XNAssets/blob/master/src/XNAssets/Assets/ResourceAssetResolver.cs) that opens Stream using Assembly.GetManifestResourceStream. Sample AssetManager creation code:
```c#
    ResourceAssetResolver assetResolver = new ResourceAssetResolver(typeof(MyGame).Assembly, "Resources.");
    AssetManager assetManager = new AssetManager(GraphicsDevice, assetResolver);
```

  * [TitleContainerAssetResolver](https://github.com/rds1983/XNAssets/blob/master/src/XNAssets/Assets/TitleContainerAssetResolver.cs) that opens Stream using TitleContainer.OpenStream. Sample AssetManager creation code:
```c#
    TitleContainerAssetResolver assetResolver = new TitleContainerAssetResolver("Assets");
    AssetManager assetManager = new AssetManager(GraphicsDevice, assetResolver);
```

# Loading Assets
After AssetManager is created, it could be used following way to load SpriteFont:
```c#
    SpriteFont font = assetManager.Load<SpriteFont>("fonts/arial64.fnt");
```
Or following way to load Texture2D:
```c#
    Texture2D texture = assetManager.Load<Texture2D>("images/LogoOnly_64px.png");
```

XNAssets allows to load following asset types out of the box:

Type|AssetLoader Type|Description
----|----------------|-----------
Texture2D|[Texture2DLoader](https://github.com/rds1983/XNAssets/blob/master/src/XNAssets/Assets/Texture2DLoader.cs)|Texture in BMP, TGA, PNG, JPG, GIF or PSD format. Alpha is being premultiplied after the loading
SpriteFont|[SpriteFontLoader](https://github.com/rds1983/XNAssets/blob/master/src/XNAssets/Assets/SpriteFontLoader.cs)|Font in AngelCode's BMFont .fnt format
string|[StringLoader](https://github.com/rds1983/XNAssets/blob/master/src/XNAssets/Assets/StringLoader.cs)|Loads any resource as string
SoundEffect|[SoundEffectLoader](https://github.com/rds1983/XNAssets/blob/master/src/XNAssets/Assets/SoundEffectLoader.cs)|SoundEffect in WAV format

# Custom Asset Types
It is possible to make XNAssets use custom asset loaders by marking custom types with attribute AssetLoaderAttribute.

I.e. following code makes it so UserProfile class will be loaded by UserProfileLoader:
```c#
    [AssetLoader(typeof(UserProfileLoader))]
    public class UserProfile
    {
        public string Name;
        public int Score;
    }
```

Now let's say that we store user profiles as xml files that look like following:
```xml
    <?xml version="1.0" encoding="utf-8" ?>
    <UserProfile>
      <Name>XNAssets</Name>
      <Score>10000</Score>
    </UserProfile>
```

Then UserProfileLoader class should look like this:
```c#
	internal class UserProfileLoader : IAssetLoader<UserProfile>
	{
		public UserProfile Load(AssetLoaderContext context, string assetName)
		{
			var data = context.Load<string>(assetName);

			var xDoc = XDocument.Parse(data);

			var result = new UserProfile
			{
				Name = xDoc.Root.Element("Name").Value,
				Score = int.Parse(xDoc.Root.Element("Score").Value)
			};

			return result;
		}
	}
```

Now it should be possible to load user profile with following code:
```c#
  UserProfile userProfile = assetManager.Load<UserProfile>("profile.xml");
```  
