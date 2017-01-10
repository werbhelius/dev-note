# 我在Android开发中遇到的坑之微博正文点击处理

> 作者：Werb

> 原文地址：http://www.jianshu.com/p/68b12336d6e0

* 开发是一个漫长的过程，我们会遇到很多很多的坑，有些却是系统级的坑，有时候遇到真是抓狂，不过这也是我们不断进步的过程，今天就给大家讲一个我遇到的一个很坑的问题。
* 还好我遇到了一个万能的 Android 大神 stainberg ，他帮助我仔细排查并且解决了问题，有他我真的提高了好多。


![QQ20161129-0@2x.png](http://upload-images.jianshu.io/upload_images/638069-d8f8c4096333498f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 需求描述

* 上图是我们常见的微博界面，其中微博正文中出现了不同标记的字段，有At用户，有##话题，有Url标签。
* 重点就是如何处理类似于微博正文中，不同标记的点击事件。
* 很显然，使用过微博SDK的同学们都知道，其中微博正文这一段字是在一个 Text 中返回的，所以我们也理应在一个 TextView 中对不同的标记做处理。
* 处理的方式很简单，就是使用 Android 中的 SpannableString 和 ClickableSpan ，先配合正则表达式匹配出想要的字符，再通过 SpannableString 的 setSpan() 方法来对标记出得字符串做处理，我们可以对该字符串自定义颜色，点击事件等（后面会有源码）。
* 注意所在的 TextView 要实现 textview.setMovementMethod(LinkMovementMethod.getInstance()) 才可以使自定义的点击事件生效。

### 一个巨大的坑
* 当我做完上面这些后，哇...好棒，每一个标记的字段都可以执行自己规定的点击事件了。
* 但是！我发现了一个很严重的问题，标记的字段是可以点击，但由于设置了 textview.setMovementMethod(LinkMovementMethod.getInstance()) 导致 TextView 对点击事件做了拦截，而原本在 RecyclerView 中 item 自己的点击事件却失效了。
* 就是说，textView 拦截了全部的点击事件，如果我这一段文字没有任何匹配到的At，##话题标签和Url这类的字符串，它任会拦截。
* 我原本想要设计的效果是，当点击特殊字符串的时候，执行自定义的点击事件，而没有特殊字符出现的时候，执行 item 原本的点击事件，例如点击正常文字，进入微博详情页。

### 排查问题
* 我想问题的原因，应该就是出在了 textview.setMovementMethod(LinkMovementMethod.getInstance()) 上面，所以我查看了 LinkMovementMethod 的源码。
* 通过打 debug 发现执行拦截操作的核心代码是下面这一段。
```java
  @Override
    public boolean onTouchEvent(TextView widget, Spannable buffer,
                                MotionEvent event) {
        int action = event.getAction();

        if (action == MotionEvent.ACTION_UP ||
            action == MotionEvent.ACTION_DOWN) {
            int x = (int) event.getX();
            int y = (int) event.getY();

            x -= widget.getTotalPaddingLeft();
            y -= widget.getTotalPaddingTop();

            x += widget.getScrollX();
            y += widget.getScrollY();

            Layout layout = widget.getLayout();
            int line = layout.getLineForVertical(y);
            int off = layout.getOffsetForHorizontal(line, x);

            ClickableSpan[] link = buffer.getSpans(off, off, ClickableSpan.class);

            if (link.length != 0) {
                if (action == MotionEvent.ACTION_UP) {
                    link[0].onClick(widget);
                } else if (action == MotionEvent.ACTION_DOWN) {
                    Selection.setSelection(buffer,
                                           buffer.getSpanStart(link[0]),
                                           buffer.getSpanEnd(link[0]));
                }

                return true;
            } else {
                Selection.removeSelection(buffer);
            }
        }

        return super.onTouchEvent(widget, buffer, event);
    }
```
* 其中有特殊字符串时，走 ```if (link.length != 0) {}```这里面，执行你的自定义点击事件，没有特使字符串的时候走 ```return super.onTouchEvent(widget, buffer, event);```
* 然后我继续对没有特使字符串的地方打断点排查，这时候我发现了一个很坑的问题，无论什么样，```return super.onTouchEvent(widget, buffer, event);```都返回 true ，这就意味着 TextView 会一直拦截事件，而外层的 item 永远不会执行点击事件，这里我终于找到了问题的所在。
* 我靠，这是一个系统级的 bug 啊，很早之前我就发现了这个问题，但我一直不知道问什么，今天终于明白了，这么久 Google 竟然还不修复。

### 解决方案
* 既然我们知道了问题出现的原因，那么就很好解决了，在没有匹配到特殊字符串的时候，返回 False 就好啦。
* 一开始我想着重写 LinkMovementMethod ，然后在最后返回 False ，然而并没有什么卵用，依旧被拦截。
* 最后在万能的 StackOverFlow 上发现了解决的方法，就是重写一个 TextView 的  setontouchlistener 方法，把上面的代码写到里面就好了，没错就是这么简单，膜拜一下 StackOverFlow 上的大神（代码如下）。
```java
  public class MyLinkMovementMethod implements View.OnTouchListener {

    public static MyLinkMovementMethod getInstance() {
        if (sInstance == null)
            sInstance = new MyLinkMovementMethod();

        return sInstance;
    }

    private static MyLinkMovementMethod sInstance;

    @Override
    public boolean onTouch(View v, MotionEvent event) {
        boolean ret = false;
        CharSequence text = ((TextView) v).getText();
        Spannable stext = Spannable.Factory.getInstance().newSpannable(text);
        TextView widget = (TextView) v;
        int action = event.getAction();

        if (action == MotionEvent.ACTION_UP ||
                action == MotionEvent.ACTION_DOWN) {
            int x = (int) event.getX();
            int y = (int) event.getY();

            x -= widget.getTotalPaddingLeft();
            y -= widget.getTotalPaddingTop();

            x += widget.getScrollX();
            y += widget.getScrollY();

            Layout layout = widget.getLayout();
            int line = layout.getLineForVertical(y);
            int off = layout.getOffsetForHorizontal(line, x);

            ClickableSpan[] link = stext.getSpans(off, off, ClickableSpan.class);

            if (link.length != 0) {
                if (action == MotionEvent.ACTION_UP) {
                    link[0].onClick(widget);
                }
                ret = true;
            }
        }
        return ret;
    }
  }
```
* 然后在 textView 上调用 ``` textView.setOnTouchListener(MyLinkMovementMethod.getInstance());```
* 就这样！有特殊字符串的地方，会执行自定义点击事件，没有特殊字符串的地方执行 item 原有的点击事件。

### 一些代码
* 其中正则表达式亲测有效，可放心使用。
```java
  /**
    * 将微博正文中的 @ 和 # ，url标识出
    *
    * @param text
    * @return
    */
   public static SpannableString getWeiBoText(Context context, String text) {
       Resources res = context.getResources();
       //四种正则表达式
       Pattern AT_PATTERN = Pattern.compile("@[\\u4e00-\\u9fa5\\w\\-]+");
       Pattern TAG_PATTERN = Pattern.compile("#([^\\#|.]+)#");
       Pattern Url_PATTERN = Pattern.compile("((http|https|ftp|ftps):\\/\\/)?([a-zA-Z0-9-]+\\.){1,5}(com|cn|net|org|hk|tw)((\\/(\\w|-)+(\\.([a-zA-Z]+))?)+)?(\\/)?(\\??([\\.%:a-zA-Z0-9_-]+=[#\\.%:a-zA-Z0-9_-]+(&)?)+)?");
       Pattern EMOJI_PATTER = Pattern.compile("\\[([\u4e00-\u9fa5\\w])+\\]");

       SpannableString spannable = new SpannableString(text);

       Matcher tag = TAG_PATTERN.matcher(spannable);
       while (tag.find()) {
           String tagNameMatch = tag.group();
           int start = tag.start();
           spannable.setSpan(new MyTagSpan(context, tagNameMatch), start, start + tagNameMatch.length(), Spannable.SPAN_INCLUSIVE_EXCLUSIVE);
       }

       Matcher at = AT_PATTERN.matcher(spannable);
       while (at.find()) {
           String atUserName = at.group();
           int start = at.start();
           spannable.setSpan(new MyAtSpan(context, atUserName), start, start + atUserName.length(), Spannable.SPAN_INCLUSIVE_EXCLUSIVE);
       }

       Matcher url = Url_PATTERN.matcher(spannable);
       while (url.find()) {
           String urlString = url.group();
           int start = url.start();
           spannable.setSpan(new MyURLSpan(context, urlString), start, start + urlString.length(), Spannable.SPAN_INCLUSIVE_EXCLUSIVE);
       }

       Matcher emoji = EMOJI_PATTER.matcher(spannable);
       while (emoji.find()) {
           String key = emoji.group(); // 获取匹配到的具体字符
           int start = emoji.start(); // 匹配字符串的开始位置
           Integer imgRes = Emotion.getImgByName(key);
           System.out.println("@@@"+imgRes);
           if (imgRes != null) {
               BitmapFactory.Options options = new BitmapFactory.Options();
               options.inJustDecodeBounds = true;
               BitmapFactory.decodeResource(res, imgRes, options);

               int scale = (int) (options.outWidth / 32);
               options.inJustDecodeBounds = false;
               options.inSampleSize = scale;
               Bitmap bitmap = BitmapFactory.decodeResource(res, imgRes, options);

               ImageSpan span = new ImageSpan(context, bitmap);
               spannable.setSpan(span, start, start + key.length(), Spannable.SPAN_EXCLUSIVE_EXCLUSIVE);
           }
       }

       return spannable;
   }

   /**
    * 用于weibo text中的连接跳转
    */
   private static class MyURLSpan extends ClickableSpan {
       private String mUrl;
       private Context context;

       MyURLSpan(Context ctx, String url) {
           context = ctx;
           mUrl = url;
       }

       @Override
       public void updateDrawState(TextPaint ds) {
           ds.setColor(Color.parseColor("#f44336"));
       }

       @Override
       public void onClick(View widget) {
           Intent intent = UrlActivity.newIntent(context, mUrl);
           context.startActivity(intent);

       }
   }
```
