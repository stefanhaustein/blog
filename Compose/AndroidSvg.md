# AndroidSvg with Jetpack Compose

**Note:** This article reflects the state in **August 2021**. Compose and Kotlin are developing rapidly, so this migth be out of date.

Currently, there doesn't seem to be a way to render SVG that is directly targeted at Jetpack Compose. So the next best thing seems to be using a View based Android SVG library such as AndroidSvg.

The most straightforward way to render AndroidSvg images inside Jetpack Compose is to wrap a SvgImageView in an AndroidView, similar to this example:

```
@Composable
fun Svg(modifier: Modifier, svg: SVG) {
    AndroidView (
        factory = { context ->
            SVGImageView(context)
        },
        modifier = modifier,
        update = { svgImageView ->
            svgImageView.setSVG(svg)
        }
    )
}
```

However, AndroidViews seem to have significant overhead. So a better way to achieve the same result is to render the SVG into a compose Canvas, similar to this:

```
@Composable
fun Svg(modifier: Modifier, svg: SVG) {
   val bounds = remember {
       mutableStateOf(RectF(0f, 0f, 48f, 48f))
   }
   Canvas(
       modifier = modifier.onGloballyPositioned {
       bounds.value = RectF(
           0f, 
           0f,
           it.size.width.toFloat(),
           it.size.height.toFloat())
        }
    ) {
        drawIntoCanvas {
            svg.renderToCanvas(
                it.nativeCanvas, bounds.value)
    }
}
```

fun Svg(modifier: Modifier, svg: SVG) {
