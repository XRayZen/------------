# customExpansion
```dart
  //既存のExpansionPanelはSliverを使えないためカスタマイズして動作を再現
  ElevatedButton customExpansion(String listTitle) {
    return ElevatedButton(
      style: ElevatedButton.styleFrom(shape: BeveledRectangleBorder()),
      onPressed: () {
        setState(() {
          _isExpanded = _isExpanded ? false : true;
        });
      },
      child: Row(
        children: <Widget>[
          const Padding(padding: EdgeInsets.all(10)),
          ExpandIcon(
            isExpanded: _isExpanded,
            onPressed: (bool isEx) {
              setState(() {
                _isExpanded = isEx ? false : true;
              });
            },
          ),
          const Padding(padding: EdgeInsets.all(10)),
          Expanded(
            child: Text(listTitle),
          ),
        ],
      ),
    );
  }
```
