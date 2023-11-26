# QRコードを表示する
>https://pub.dev/packages/qr_flutter/install
>https://www.kamo-it.org/blog/36/

## インストール
- `flutter pub add qr_flutter`
```yaml
dependencies:
  qr_flutter: ^4.0.0
```
## 使い方
```dart
import 'package:qr_flutter/qr_flutter.dart';

QrImage(
  data: "https://flutter.dev",
  version: QrVersions.auto,
  size: 200.0,
  gapless: false,
  embeddedImage: AssetImage('assets/flutter-mark-square-64.png'),
  embeddedImageStyle: QrEmbeddedImageStyle(
    size: Size(40, 40),
  ),
),
```
## 読み取る
>https://pub.dev/packages/qr_code_scanner
- `flutter pub add qr_code_scanner`
## 権限の設定
- `android/app/src/main/AndroidManifest.xml`
```xml
<uses-permission android:name="android.permission.CAMERA" />
```
- `ios/Runner/Info.plist`
```xml
<key>NSCameraUsageDescription</key>
<string>カメラを使用してQRコードを読み取ります</string>
```

## 使い方
```dart
import 'dart:developer';
import 'dart:io';

import 'package:flutter/material.dart';
import 'package:qr_code_scanner/qr_code_scanner.dart';

class QrScanView extends StatefulWidget {
  const QrScanView({Key? key}) : super(key: key);

  @override
  _QrScanViewState createState() => _QrScanViewState();
}

class _QrScanViewState extends State<QrScanView> {
  final GlobalKey qrKey = GlobalKey(debugLabel: 'QR');
  QRViewController? controller;

  // In order to get hot reload to work we need to pause the camera if the platform
  // is android, or resume the camera if the platform is iOS.
  @override
  void reassemble() {
    super.reassemble();
    if (Platform.isAndroid) {
      controller?.pauseCamera();
    }
    controller?.resumeCamera();
  }

  @override
  void dispose() {
    controller?.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
          appBar: AppBar(
            title: Text('QR読み取り'),
            centerTitle: true,
          ),
          body: _buildQrView(context),
        );
  }

  Widget _buildQrView(BuildContext context) {
    // For this example we check how width or tall the device is and change the scanArea and overlay accordingly.
    var scanArea = (MediaQuery.of(context).size.width < 400 ||
            MediaQuery.of(context).size.height < 400)
        ? 150.0
        : 300.0;
    // To ensure the Scanner view is properly sizes after rotation
    // we need to listen for Flutter SizeChanged notification and update controller
    return QRView(
      key: qrKey,
      onQRViewCreated: _onQRViewCreated,
      overlay: QrScannerOverlayShape(
          borderColor: Colors.red,
          borderRadius: 10,
          borderLength: 30,
          borderWidth: 10,
          cutOutSize: scanArea),
      onPermissionSet: (ctrl, p) => _onPermissionSet(context, ctrl, p),
    );
  }

  void _onQRViewCreated(QRViewController controller) {
    setState(() {
      this.controller = controller;
    });
    controller.scannedDataStream.listen((scanData) async {
      print(scanData.code);
    });
  }

  void _onPermissionSet(BuildContext context, QRViewController ctrl, bool p) {
    log('${DateTime.now().toIso8601String()}_onPermissionSet $p');
    if (!p) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('no Permission')),
      );
    }
  }
}
```









