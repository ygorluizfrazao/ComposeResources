# compose-resources
Simple classes to help dealing with resources in a decoupled and reusable way.

<div id="header" align="center">
  <a href="https://jitpack.io/#ygorluizfrazao/highlighted-text-compose"><img src="https://jitpack.io/v/ygorluizfrazao/highlighted-text-compose.svg" alt="Version Name"/></a>
  <img src="https://komarev.com/ghpvc/?username=ygorluizfrazao&style=flat-square&color=blue" alt=""/>
</div>
<div id="badges" align="center">
  <a href="https://www.linkedin.com/in/ygorluizfrazao/">
    <img src="https://img.shields.io/badge/LinkedIn-blue?style=flat&logo=linkedin&logoColor=white" alt="LinkedIn Badge"/>
  </a>
  <a href="https://ko-fi.com/ygorfrazao">
    <img src="https://img.shields.io/badge/Kofi-blue?style=flat&logo=kofi&logoColor=white" alt="Youtube Badge"/>
  </a>
</div>

## How can i use it?

Just add this to your *settings.gradle*:

```groovy
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
        maven { url 'https://jitpack.io' }
    }
}
```

Then, in your *build.gradle*:

```groovy
	dependencies {
  
	}
```

## What it does?

It provides to classes aimed to help you use String and Image resources in a more decoupled way.

### Icon Resource

Sometimes we need flexibility between using an ImageVector or a Drawable Resource in Android, one such case would be when you need different icons according to a [navigation destination]([https://developer.android.com/guide/navigation?gclid=Cj0KCQjwiZqhBhCJARIsACHHEH_PTv5kYGJrxhDS_lN8-puXUyfGclFI89hXe7FZHcxIE3HvfjZ1gFkaApm9EALw_wcB&gclsrc=aw.ds]), such as:

```kotlin
sealed class Screen(
    val route: TextResource,
    val localizedAlias: TextResource,
    val icon: IconResource
) {
    object NotesList : Screen(
        TextResource.StringResource(R.string.nav_notes_list),
        TextResource.StringResource(R.string.notes_list),
        IconResource.fromImageVector(Icons.Filled.List)
    )

    object Bin :
        Screen(
            TextResource.StringResource(R.string.nav_notes_bin),
            TextResource.StringResource(R.string.bin),
            IconResource.fromImageVector(Icons.Filled.Delete)
        )
}
```

In this class definition we have the flexibility to use both `Icons` image vectors or our custom drawable, provided the functions `fromDrawableResource` and `fromImageVector`.

If you need an `Icon` composable, as its needed inside an `IconButton` function, you can call `ComposeIcon()` function, or, if you just need a `PainterResource`, call `asPainterResource()`.

The class is implemented as follows:

```kotlin
class IconResource private constructor(
    @DrawableRes private val resID: Int?,
    private val imageVector: ImageVector?,
    val contentDescription: String
) {

    @Composable
    fun asPainterResource(): Painter {
        resID?.let {
            return painterResource(id = resID)
        }
        return rememberVectorPainter(image = imageVector!!)
    }

    @Composable
    fun ComposeIcon(modifier: Modifier = Modifier) {
        Icon(
            modifier = modifier,
            painter = asPainterResource(),
            contentDescription = contentDescription
        )
    }

    companion object {
        @JvmStatic
        @SuppressLint
        fun fromDrawableResource(
            @DrawableRes resID: Int,
            contentDescription: String = ""
        ): IconResource {
            return IconResource(resID, null, contentDescription)
        }

        @JvmStatic
        fun fromImageVector(
            imageVector: ImageVector?,
            contentDescription: String = ""
        ): IconResource {
            return IconResource(null, imageVector, contentDescription)
        }
    }
}
```

### Text Resource

Mostly, abstracts the string.xml values, you will not need a context inside you viewmodel to convert a string resource to a real text. Create the apropriate `TextResource` class and call `asString()`when you need.

And example of its usage would be:

```kotlin

_uiState.value = UIState.TitleError(
    titleInvalidError = TextResource.StringResource(
	R.string.invalid_title,
	"$minLength"
    )
)

//Then, in your ui
titleInvalidError.asString()

```

Class implementation be like:

```kotlin
sealed class TextResource {

    data class RuntimeString(val value: String) : TextResource()

    class StringResource(
        @StringRes val resId: Int,
        vararg val args: Any
    ) : TextResource()


    @Composable
    fun asString(): String {
        return when (this) {
            is RuntimeString -> value
            is StringResource -> stringResource(id = resId, *args)
        }
    }

    @SuppressLint
    fun asString(context: Context): String {
        return when (this) {
            is RuntimeString -> value
            is StringResource -> context.getString(resId, *args)
        }
    }
}
```

Hope it helps you.
