---
title: "ReactのモーダルフォームをNetlifyのFormsで使う"
emoji: "💡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["netlify", "react", "bootstrap"]
published: true
---
# はじめに

react-bootstrapのModal上に配置したFormをNetlifyのFormsで使おうとした時にハマりました。
同じようなことでつまずいている人の参考になれば。

## 何にハマったか?

NetlifyのFormsは、`<form name="contact" method="POST" data-netlify="true">` と記述したformを作ると、デプロイ時に解析してそこでsubmitされた内容を管理画面に表示してくれる便利機能です。
ReactのGatsby.jsで画面を作っていて、react-bootstrapのモーダルで入力フォームを表示して、そこでNetlifyのFormsを使おうとしました。

が、しかし。
モーダルはデプロイ時にはHTMLの要素として作成されておらず、ボタンを押して表示するタイミングでHTML要素が動的に追加されます。
このため、デプロイ時にはフォームがなく、NetlifyのFormsが使えませんでした。

# 対策

これが正解という自信は全くありませんが、私が考えた対策は、bootstrapで表示するモーダル内のフォーム以外にもう一つ非表示でNetlify用のフォームを作ることでした。
:::message
もし他にもっとスマートな方法がある場合は是非教えてください。
よろしくお願いします。🙏
:::

# コード例

以下、具体的なコード例です。色々端折っていますので、そのままコピペでは動かないです。

## ダミーフォーム部分

```jsx:Netlify Formsに認識させる部分(ダミーフォーム)
<div style={{ display: 'none' }}>
  <form name="contact" method="POST" data-netlify="true">
    <input type="hidden" name="form-name" value="contact" />
    <input type="text" name="formname" defaultValue={formname} />
    <input type="text" name="formemail" defaultValue={formemail} />
  </form>
</div>
```
- styleで非表示にします。
`type="hidden"`だと内容が送信されません。
- ここで定義したフィールドのみ送信されます。
- submitボタンで送信ではなく、axiosでPOSTします。
submitの場合一旦Netlifyの画面に遷移するので、それを避けるため。
- 入力フィールドは書込みが発生するので、`defaultValue`にする。

こうすることで、Formsにフォームがあることを認識させます。

## モーダル部分

```jsx:モーダル部分
<Button onClick={openModal}>モーダルを開く</Button>

<Modal show={modalIsOpen} onHide={() => closeModal()} aria-labelledby="modal-dialog">
  <Modal.Header closeButton bsPrefix="modal-header">
    <Modal.Title id="modal-dialog">モーダルフォーム</Modal.Title>
  </Modal.Header>
  <Modal.Body>
    <Form onSubmit={handleSubmit}>
      <Form.Group controlId="exampleForm.ControlInput1">
        <Form.Label>お名前</Form.Label>
        <Form.Control type="text" name="name" value={formname} onChange={(e) => setFormName(e.target.value)} />
      </Form.Group>
      <Form.Group controlId="exampleForm.ControlInput2">
        <Form.Label>連絡先(email)</Form.Label>
        <Form.Control size="lg" type="email" name="email" value={formemail} onChange={(e) => setFormEmail(e.target.value)} />
      </Form.Group>
      <Button type="submit">送信する</Button>
    </Form>
  </Modal.Body>
</Modal>
```

- 「モーダルを開く」ボタンをクリックすることでモーダルを表示します。
- 入力フィールドは`onChange`のイベントを拾い、`formname/formemail`を更新します。
- `submit`ボタンをクリックすることで`handleSubmit`を呼び出します。

## import
```jsx:import
import React, { useState } from 'react';
import axios from 'axios';
import { Modal, Form, Button } from 'react-bootstrap';
```

ライブラリは以下を使用しています。
```json:package.json
"dependencies": {
  "axios": "^0.21.1",
  "react-bootstrap": "^1.0.1",
  ...
}
```

## Hook部分

```jsx:hookとモーダルopen/close
const [modalIsOpen, setIsOpen] = useState(false);
const [formname, setFormName] = useState('');
const [formemail, setFormEmail] = useState('');

const openModal = () => {
  setIsOpen(true);
};
const closeModal = () => {
  setIsOpen(false);
  setFormName('');
  setFormEmail('');
};
```

## handleSubmit部分

```jsx:handleSubmitでダミーフォーム入力内容を送信
const handleSubmit = (event) => {
  event.currentTarget.preventDefault();
  const params = new URLSearchParams();
  params.append('form-name', 'contact');
  params.append('formname', formname);
  params.append('formemail', formemail);
  axios
    .post('/', params)
    .then(() => {
      console.log('送信に成功しました');
    })
    .catch((err) => {
      console.log(`送信に失敗しました: ${err}`);
    });
};
```

# さいごに

このようにダミーフォームを作り、モーダル内の入力情報を転送してやることでFormsが使えるようになります。

Netlifyは便利機能を備えていて無料でもそれなりに使いやすいと思います。
私自身が、まだまだ機能を十分引き出せていないと感じています。
今後も引き続き様々な機能を試して、知見を得れば公開していきます。
