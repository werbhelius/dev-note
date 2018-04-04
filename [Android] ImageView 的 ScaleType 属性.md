# Android ImageView 的 ScaleType 属性

ImageView 的默认的 ScaleType 是 fitCenter

1. maxtrix : 图片会从左上角起，显示到 ImageView 中，不做任何缩放，如果矩形比图形大，则图片显示在左上角，如果矩形比图形小，则只会显示图形矩形部分的大小。
2. fitCenter : 图片完全显示，将图片等比缩放，并显示在 ImageView 的中间，确保图片会显示完全，缩放时会充满矩形的短边。
3. fixXY : 图片完全显示，并缩放的矩形的大小
4. fitStart	: 图片完全显示 与 fitCenter 缩放方式一样，只是显示在 ImageView 的左边
5. fitEnd : 图片完全显示 与 fitCenter 缩放方式一样，只是显示在 ImageView 的右边
6. center : 不做任何缩放操作，将图片按照原来的大小居中显示，超出ImageView大小部分被截断，注意是从两边等分截断。如果图片大小小于ImageView大小，则居中显示，图片可能部分显示。
7. centerCrop : 将图片按照等比例缩放，并截取缩放后的中间部分显示在ImageView中。（使得图片的高等于View的高，使得图片宽等于或大于View的宽）（图片可能部分显示）
8. centerInside : 将图片大小大于ImageView的图片进行等比例缩小，直到整幅图能够居中显示在ImageView中，小于ImageView的图片不变，直接居中显示。（图片完整显示）

centerInside和fitCenter最主要的区别是，当ImageView大小大于图片大小时候，centerInside直接显示图片原大小，而fitCenter，则会放大图片，使得图片能够充满矩形的短边。 