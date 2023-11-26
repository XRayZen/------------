
>https://gakogako.com/flutter_elevated_button/

## ElevatedButtonの定義・枠線・リップル
```dart
ElevatedButton(
  style: ElevatedButton.styleFrom(
    //リップルエフェクトだが標準でセットされている
    splashFactory: InkRipple.splashFactory,
    side: BorderSide(

      color: Colors.red, //色
      width: 5, //太さ
    ),
  ),
  onPressed: () {},
  child: Text('Button'),
),
```