

# Import
```typescript
import {
  Drawer,
  DrawerBody,
  DrawerFooter,
  DrawerHeader,
  DrawerOverlay,
  DrawerContent,
  DrawerCloseButton,
} from '@chakra-ui/react'
```


# useDisclosureとは
GitHub Copilot: useDisclosure()は、Chakra UIのフックの1つで、Drawer、Modal、PopoverなどのUIコンポーネントで使用される状態を管理するために使用されます。このフックは、isOpen、onOpen、onClose、onToggleという4つのプロパティを返します。

isOpenは、Drawer、Modal、PopoverなどのUIコンポーネントが開いているかどうかを示す真偽値です。onOpenは、UIコンポーネントを開くための関数で、onCloseは、UIコンポーネントを閉じるための関数です。onToggleは、UIコンポーネントを開く/閉じるためのトグル関数です。

以下は、useDisclosure()フックを使用して、Drawerコンポーネントを制御する例です。

```typescript
import { useDisclosure } from "@chakra-ui/react";
import { Drawer, DrawerOverlay, DrawerContent, DrawerCloseButton, DrawerHeader, DrawerBody } from "@chakra-ui/react";

function MyDrawer() {
  const { isOpen, onOpen, onClose } = useDisclosure(); // useDisclosure()フックを使用して、isOpen、onOpen、onCloseを定義
  return (
    <>
      <button onClick={onOpen}>Open Drawer</button> {/* ボタンをクリックするとDrawerを開く */}
      <Drawer isOpen={isOpen} onClose={onClose}>
        <DrawerOverlay />
        <DrawerContent>
          <DrawerCloseButton />
          <DrawerHeader>Drawer Header</DrawerHeader>
          <DrawerBody>Drawer Body</DrawerBody>
        </DrawerContent>
      </Drawer>
    </>
  );
}
```

この例では、useDisclosure()フックを使用して、isOpen、onOpen、onCloseを定義しています。ボタンをクリックすると、onOpen関数が呼び出されてDrawerが開きます。DrawerコンポーネントのisOpenプロパティにisOpenを渡し、onCloseプロパティにonCloseを渡しています。これにより、Drawerコンポーネントを制御することができます。


```typescript
const DrawerMenu = () => {
  // useDisclosureで閉じ・開きの管理
  const { isOpen, onOpen, onClose } = useDisclosure()
  const btnRef = React.useRef<HTMLButtonElement>(null)

  return (
    <>
      {/* ハンバーガーアイコン部分 */}
      <Button ref={btnRef} onClick={onOpen}>
        <HamburgerIcon />
      </Button>
      {/* Drawer部分 */}
      <Drawer
        isOpen={isOpen}
        onClose={onClose}
        placement="left"
        finalFocusRef={btnRef}
      >
        <DrawerOverlay>
          <DrawerContent>
            <DrawerCloseButton />
            <DrawerHeader>
              Menu
            </DrawerHeader>
            <DrawerBody>
              <Navigation />　{/*　先程作ったNavigatorを利用 */}
            </DrawerBody>
          </DrawerContent>
        </DrawerOverlay>
      </Drawer>
    </>
  )
```

# 2 方向をボタンで指定する
```typescript
"use client";
import {
  Drawer,
  DrawerBody,
  DrawerFooter,
  DrawerHeader,
  DrawerOverlay,
  DrawerContent,
  DrawerCloseButton,
  Button,
  Radio,
  RadioGroup,
  Stack,
  useDisclosure,
} from '@chakra-ui/react';
import React from 'react';

type DrawerPlacement = 'left' | 'right' | 'top' | 'bottom';

export default function DrawerMenu() {
  const { isOpen, onOpen, onClose } = useDisclosure();
  const [placement, setPlacement] = React.useState('right');

  return (
    <>
      <RadioGroup defaultValue={placement} onChange={setPlacement}>
        <Stack direction='row' mb='4'>
          <Radio value='top'>Top</Radio>
          <Radio value='right'>Right</Radio>
          <Radio value='bottom'>Bottom</Radio>
          <Radio value='left'>Left</Radio>
        </Stack>
      </RadioGroup>
      <Button colorScheme='blue' onClick={onOpen} textColor='white'>
        Open
      </Button>
      <Drawer placement={convertDrawerMenuPlacement(placement)} onClose={onClose} isOpen={isOpen}>
        <DrawerOverlay />
        <DrawerContent>
          <DrawerHeader borderBottomWidth='1px'>Basic Drawer</DrawerHeader>
          <DrawerBody>
            <p>Some contents...</p>
            <p>Some contents...</p>
            <p>Some contents...</p>
          </DrawerBody>
        </DrawerContent>
      </Drawer>
    </>
  );
}

// 引数のストリングがtopだったら、返すDrawerのplacementはtopになる
function convertDrawerMenuPlacement(placement: string): DrawerPlacement {
  if (placement === 'top') {
    return 'top';
  } else if (placement === 'right') {
    return 'right';
  } else if (placement === 'bottom') {
    return 'bottom';
  } else if (placement === 'left') {
    return 'left';
  } else {
    return 'bottom';
  }
}
```
