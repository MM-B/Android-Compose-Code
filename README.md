# Using Jetpack Compose In Android Applications

In this short read me, I want to show you how I'm using Jetpack Compose in my applications and how I handle different thing with Jetpack
Compose.

[Theming in Jetpack Compose](#theming)

[Navigation In Jetpack Compose](#navigation)

[Atomic Design](#atomic)

[Simple Page In Compose](#page)
<a name="theming"/>

## Theming In Jetpack Compose

Jetpack Compose lets us fully customize the theme and design system. With help of the material library, I access Material Theme and I could
easily initialize it with my own information.

If I need more, I could extend the Material theme by using some extension functions:

```kotlin
val Colors.brand: Color
    get() = if (isLight) RedList else RedDark
```

But big project always has their own design systems that include colors, shapes, dimensions, and topographies. In this situation, I have to
create my own custom theme with Jetpack Compose.

Then I have to create my own theme including colors, topographies, and dimensions. As an example:

```kotlin

class ProjectColors(
    brand: Color,
    isDark: Boolean,
)
```

```kotlin

class ProjectTypographies(
    val header: TextStyle,
)
```

After creating classes for my own design system I have to create Composition Locals.

```kotlin
val LocalColors = staticCompositionLocalOf<EProejctColors> {
    error("No TestColorPalette provided")
}

```

Then I could create my own theme.

```kotlin
object ProjectTheme {
    val colors: ProjectColors
        @Composable
        get() = LocalColors.current

    val typography: ProjectTypography
        @Composable
        get() = LocalTypography.current
}
```

And finally, use it in the app.

```kotlin

@Composable
fun ProjectTheme(
    isDarkTheme: Boolean = isSystemInDarkTheme(),
    content: @Composable () -> Unit
) {
    val projectColors = getProjectColorsValues(isDarkTheme)
        .CompositionLocalProvider(
                LocalColors provides projectColors,
        ) {
            content()
        }
}

```

But in some cases I want to use different colors or dimensions for different screen sizes, then I use:

```kotlin
internal fun <T> getDeviceScreenStyleValues(
    configuration: Configuration,
    screens: Screens<T>
): T {
    return when (configuration.screenWidthDp) {
        in 0..480 -> screens.small()
        in 480..720 -> screens.medium()
        else -> screens.large()
    }
}
```

And based on screen sizes I return different initialized objects of colors, dimensions, or...

```kotlin
internal object TypographyValues : Screens<ProjectTypographies> {

    override fun small() = ProjectTypographies(
            header = TextStyle(
                    fontSize = 24.0.sp,
            )
    )

    override fun medium() = ProjectTypographies(
            header = TextStyle(
                    fontSize = 25.0.sp,
            )
    )

    override fun large() = ProjectTypographies(
            header = TextStyle(
                    fontSize = 26.0.sp,
            )
    )

```

<a name="navigation"/>

## Navigation In Jetpack Compose

For handling navigation in Compose, I declare each graph and the screens that are inside of that.

I have navHost and I declare graphs in it.

```kotlin
 NavHost(
        navController = navController,
        startDestination = SplashGraph.route,
) {

    homeGraph(
            navControler = navController,
            navigateToSomePage = {
                navController.navigate(AnotherGraph.Logout.route)
            }
    )
}

```

I add screens inside graphs.

```kotlin
fun NavGraphBuilder.homeGraph(
    navController: NavController,
    navigateToSomePage: () -> Unit,
) {
    navigation(
            route = HomeGraph.route,
            startDestination = HomeGraph.Login.route,
    ) {
        loginScreen(
                navcontroler = navcontroller,
                navigateToSomePage = navigateToSomePage,
        )
    }
```

I add composable screens.

```kotlin
private fun NavGraphBuilder.loginScreen(
    navController: NavController,
    navigateToSomePage: () -> Unit,
) {
    composable(
            screen = HomeGraph.Login,
            arguments = HomeGraph.Login.navArguments,
    ) {
        LoginScreen(
                navigateToSomePage = navigateToSomePage,
                navigateToOtherPageInGraph = {
                    navController.navigate(HomeGraph.Success.route)
                }
        )
    }
}

```

Also, I create graphs objects to access better.

```kotlin
interface Graph {
    val route: String
}

interface Screen {
    val graph: Graph
    val route: String
    var navArguments: List<NamedNavArgument>

    val absoluteRoute: String
        get() = "${graph.route}/$route"
}

object HomeGraph : Graph {
    override val route: String = "home"

    object Login : Screen(
            route = "login/$KEY",
            graph = HomeGraph,
            navArguments = listOf(
                    navArgument(KEY) { type = NavType.IntType },
            )
    ) {
        fun createRoute(number: Int) = "$route/$number"
    }
}
```

.
<a name="atomic"/>

## Atomic Design

Because Jetpack Compose gives us the ability to create reusable components along with the UI, I decided to choose a pattern for creating and
using reusable UI elements. And I use them for my UI elements across the application. this helps us to easily do any changes across the app
by editing only one main composable.

By using that I have atoms, molecules, and organisms. Atoms are made up of pure compose functions. Molecules are made up of combined atoms.
Organisms are made up of combined molecules. (also they could handle state)

As an example for text fields:

Atom:

```kotlin
@Composable
fun ProjectBasicTextField(
    value: String,
    onValueChange: (String) -> Unit,
    modifier: Modifier = Modifier,
    enabled: Boolean = true,
    readOnly: Boolean = false,
    textStyle: TextStyle = TextStyle.Default,
    keyboardOptions: KeyboardOptions = KeyboardOptions.Default,
    keyboardActions: KeyboardActions = KeyboardActions.Default,
    singleLine: Boolean = false,
    maxLines: Int = Int.MAX_VALUE,
    visualTransformation: VisualTransformation = VisualTransformation.None,
    onTextLayout: (TextLayoutResult) -> Unit = {},
    interactionSource: MutableInteractionSource = remember { MutableInteractionSource() },
    cursorBrush: Brush = SolidColor(AppTheme.colors.systemBlack),
    selectionBackgroundColor: Color = AppTheme.colors.systemBlack.copy(.3f),
    decorationBox: @Composable (innerTextField: @Composable () -> Unit) -> Unit = @Composable { innerTextField -> innerTextField() }
) {
    CompositionLocalProvider(
            LocalTextSelectionColors provides TextSelectionColors(
                    handleColor = Color.Transparent,
                    backgroundColor = selectionBackgroundColor,
            )
    ) {
        BasicTextField(
                value = valye,
                onValueChange = {
                    onValueChange(it)
                },
                modifier = modifier,
                enabled = enabled,
                readOnly = readOnly,
                textStyle = textStyle,
                keyboardOptions = keyboardOptions,
                keyboardActions = keyboardActions,
                singleLine = singleLine,
                maxLines = maxLines,
                visualTransformation = visualTransformation,
                onTextLayout = onTextLayout,
                interactionSource = interactionSource,
                cursorBrush = cursorBrush,
                decorationBox = decorationBox,
        )
    }
}
```

Molecule:

```kotlin
@Composable
fun ProjectTextField(
    value: String,
    onValueChange: (String) -> Unit,
    modifier: Modifier = Modifier,
    visualTransformation: VisualTransformation = VisualTransformation.None,
    errorMessage: String = "",
    placeholder: String = "",
    isError: Boolean = false,
    enabled: Boolean = true,
    readOnly: Boolean = false,
    singleLine: Boolean = true,
    textStyle: TextStyle = AppTheme.typography.subtitle,
    placeHolderStyle: TextStyle = textStyle.copy(color = AppTheme.colors.systemGrey),
    @DrawableRes
    leadingIcon: Int? = null,
    leadingIconContentDescription: String = "ProjectTextField",
    onLeadingIconClicked: () -> Unit = {},
    @DrawableRes
    trailingIcon: Int? = null,
    trailingIconContentDescription: String = "projectTextField",
    onTrailingIconClicked: () -> Unit = {},
    colors: TextFieldColors = projectTextFieldColors(),
    showClearButton: Boolean = true,
) {
    ProjectBasicTextField(
            value = value,
            textStyle = AppTheme.typography.subtitle,
            onValueChange = onValueChange,
            visualTransformation = visualTransformation,
            readOnly = readOnly,
            singleLine = singleLine,
            decorationBox = { coreTextField ->
                TextFieldDecoration(
                        colors = colors,
                        enabled = enabled,
                        leadingIcon = leadingIcon,
                        onLeadingIconClicked = onLeadingIconClicked,
                        leadingIconContentDescription = leadingIconContentDescription,
                        value = value,
                        placeholder = placeholder,
                        placeHolderStyle = placeHolderStyle,
                        coreTextField = coreTextField,
                        trailingIcon = trailingIcon,
                        onTrailingIconClicked = onTrailingIconClicked,
                        trailingIconContentDescription = trailingIconContentDescription,
                        isError = isError,
                        errorMessage = errorMessage,
                        textStyle = textStyle,
                        showClearButton = showClearButton,
                        onClearIconClicked = { onValueChange("") },
                )
            },
            selectionBackgroundColor = colors.cursorColor(isError = isError).value.copy(alpha = 0.4f),
            modifier = Modifier
                .fillMaxWidth()
                .then(modifier)

    )
}

```

.
<a name="page"/>

## Simple Page In Compose

This is a simple Compose page.

```kotlin

@Composable
fun SampleScreen(
    navigateToLogin: () -> Unit,
    viewModel: SampleViewModel = hiltViewModel(),
) {
    val coroutineScope = rememberCoroutineScope()

    SideEffect {
        viewModel.container.sideEffectFlow
            .onEach { sideEffect ->
                when (sideEffect) {
                    is SampleSideEffect.navigateToLogin -> navigateToLogin()
                }
            }
            .launchIn(coroutineScope)
    }

    val state by viewModel.container.stateFlow.collectAsState()

    SampleScreen(
            state = state,
            onCloseClicked = {
                viewModel.dispatchIntent(SampleIntent.CloseClicked)
            },
    )
}

@Composable
fun SampleScreen(
    state: PageViewState,
    onCloseClicked: () -> Unit,
) {
    Column(
            modifier = Modifier
                .fillMaxWidth()
                .fillMaxHeight(0.9f)
                .background(AppTheme.colors.systemWhite)
                .verticalScroll(rememberScrollState()),
    ) {
        Header(onCloseClicked)

        Spacer(
                modifier = Modifier
                    .height(AppTheme.dimensions.XXX_SMALL_SPACE)
        )

        Subtitle(
                displaySubtitle = state.displaySubtitle,
                subtitle = state.subtitle,
        )

        OfferInformation(
                openingDays = state.openingDays,
                openingTime = state.openingTime,
        )

        Spacer(
                modifier = Modifier
                    .height(AppTheme.dimensions.X_NORMAL_SPACE)
        )

        LoyaltyInformation(
                name = state.loyalatyName,
                address = state.loyaltyAddress,
                logo = state.loyaltyLogo,
        )

        Spacer(
                modifier = Modifier
                    .height(AppTheme.dimensions.X_SMALL_SPACE)
        )

        ProjectImage(
                uri = state.mapImage,
                contentDescription = stringResource(R.string.map_content_dscription),
                contentScale = ContentScale.Crop,
                modifier = Modifier
                    .padding(horizontal = AppTheme.dimensions.NORMAL_SPACE)
                    .fillMaxWidth()
                    .aspectRatioHeight(.4f)
                    .clip(AppTheme.shapes.roundedCorner10)
                    .border(
                            width = AppTheme.dimensions.BUTTON_BORDER_SIZE,
                            shape = AppTheme.shapes.roundedCorner10,
                            color = AppTheme.colors.systemExtraLightGrey
                    )
        )

        Spacer(
                modifier = Modifier
                    .height(AppTheme.dimensions.XXX_LARGE_SPACE)
        )
    }
}

@Composable
private fun Header(onSomeThingClicked: () -> Unit) {
    Row(
            modifier = Modifier
                .fillMaxWidth()
                .padding(AppTheme.dimensions.X_NORMAL_SPACE),
    ) {
        ProjectText(
                text = state.title,
                style = AppTheme.typography.h2,
                modifier = Modifier
                    .weight(1f)
        )

        IconButton(
                iconResourceId = R.drawable.ic_close,
                onClick = onSomeThingClicked,
        )
    }
}

@Composable
private fun Subtitle(
    displaySubtitle: Boolean,
    subtitle: String,
) {
    if (displaySubtitle)
        ProjectText(
                text = subtitle,
                style = AppTheme.typography.body2,
                modifier = Modifier
                    .padding(horizontal = AppTheme.dimensions.NORMAL_SPACE)
        )
}

@Composable
private fun OfferInformation(
    openingDays: String,
    openingTime: String,
) {
    Column(
            modifier = Modifier
                .background(AppTheme.colors.systemOffWhite)
                .padding(horizontal = AppTheme.dimensions.NORMAL_SPACE),
            horizontalAlignment = Alignment.CenterHorizontally,
    ) {
        Spacer(
                modifier = Modifier
                    .height(AppTheme.dimensions.X_NORMAL_SPACE)
        )

        ProjectText(
                text = stringResource(R.string.available_on),
                style = AppTheme.typography.body1,
        )

        Spacer(
                modifier = Modifier
                    .height(AppTheme.dimensions.SMALL_SPACE)
        )

        OpeningDaysAndTime(
                openingDays = openingDays,
                openingTime = openingTime,
        )

        ProjectText(
                text = stringResource(R.string.reward_claim_rule),
                style = AppTheme.typography.body1,
                textAlign = TextAlign.Center,
                modifier = Modifier
                    .clip(AppTheme.shapes.roundedButton)
                    .background(AppTheme.colors.systemWhite)
                    .padding(horizontal = AppTheme.dimensions.XX_SMALL_SPACE)
                    .weight(1f),
        )
    }
}

@Composable
private fun OpeningDaysAndTime(
    openingDays: String,
    openingTime: String
) {
    Row {
        ProjectText(
                text = openingDays,
                style = AppTheme.typography.body1,
                textAlign = TextAlign.Center,
                modifier = Modifier
                    .clip(AppTheme.shapes.roundedButton)
                    .background(AppTheme.colors.systemWhite)
                    .weight(1f),
        )

        Spacer(
                modifier = Modifier
                    .width(AppTheme.dimensions.XX_SMALL_SPACE)
        )

        ProjectText(
                text = openingTime,
                style = AppTheme.typography.body1,
                textAlign = TextAlign.Center,
                modifier = Modifier
                    .clip(AppTheme.shapes.roundedButton)
                    .background(AppTheme.colors.systemWhite)
                    .padding(horizontal = AppTheme.dimensions.XX_SMALL_SPACE)
                    .weight(1f),
        )
    }
}

@Composable
private fun LoyaltyInformation(
    name: String,
    address: String,
    logo: String,
) {
    Row(
            modifier = Modifier
                .padding(horizontal = AppTheme.dimensions.NORMAL_SPACE)
                .fillMaxWidth(),
            verticalAlignment = Alignment.CenterVertically
    ) {
        ProjectImage(
                uri = logo,
                contentDescription = stringResource(R.string.loyalty_logo_content_description),
                contentScale = ContentScale.Crop,
                modifier = Modifier
                    .clip(AppTheme.shapes.roundedButton)
                    .size(AppTheme.dimensions.XX_LARGE_SPACE)
        )

        Column(
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(horizontal = AppTheme.dimensions.XX_SMALL_SPACE)
        ) {
            ProjectText(
                    text = name,
                    style = AppTheme.typography.body1,
            )

            ProjectText(
                    text = address,
                    style = AppTheme.typography.body2,
                    color = AppTheme.colors.systemDarkGrey,
            )
        }
    }
}

@Preview(device = Devices.PIXEL_4)
@Preview(device = Devices.PIXEL_4_XL)
@Composable
fun OfferDetailPreview() {
    ProjectTheme {
        SampleScreen(
                state = PageViewState(),
                onCloseClicked = {},
        )
    }
}
```
