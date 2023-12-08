实现svg图片显示时的BoxFit.contain效果
```dart
  Path getClip(Size size) {
    var path = parseSvgPathData(svgPath);
    // 计算宽度和高度的缩放因子
    double widthScaleFactor = size.width / svgSize.width;
    double heightScaleFactor = size.height / svgSize.height;

    // 选择较小的缩放因子
    double scaleFactor = min(widthScaleFactor, heightScaleFactor);

    final Matrix4 matrix4 = Matrix4.identity();
    matrix4.scale(scaleFactor, scaleFactor);

    // 可选 - 计算居中的偏移量
    double dx = (size.width - svgSize.width * scaleFactor) / 2;
    double dy = (size.height - svgSize.height * scaleFactor) / 2;

    return path.transform(matrix4.storage).shift(Offset(dx, dy));
  }
```