# HTMLコンテンツをFlutterWidgetとして表示
>https://blog.pentagon.tokyo/2293/

1. install
`flutter pub add flutter_html`

## example
```dart
Widget html = Html(
 data: htmlText,
 //フォントサイズ
 style: {
        'body': Style(
          fontSize: FontSize(
            getResponsiveValue(
              context,
              defaultValue: 15,
              mobileValue: 18,
              tabletValue: 14,
            ),
          ),
        )
      },
);
```



