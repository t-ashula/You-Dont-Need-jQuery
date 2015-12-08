## You Don't Need jQuery

Web のフロントエンド環境は日々進化し，十分で大量な DOM/BOM API がすでに主要なブラウザにはある．DOM やイベントの処理のために jQuery を初めから学ぶ必要はない．その一方で，ありがたいことに React や Angular, Vue と言ったライブラリの流行で DOM を直接触るのはアンチパターンとなりつつあり，jQuery はより重要なものでなくなっている．このプロジェクトでは IE10 以上で使えるネイティブな代替実装についてまとめておく．

## もくじ

1. [Query Selector](#query-selector)
1. [CSS & Style](#css-style)
1. [DOM Manipulation](#dom-manipulation)
1. [Ajax](#ajax)
1. [Events](#events)
1. [Utilities](#utilities)
1. [Translation](#translation)
1. [Browser Support](#browser-support)

## Query Selector

クラスや ID，属性と言った一般的なセレクタには `document.querySelector` か `document.querySelectorAll` が代わりに使える．
違いは
* `document.querySelector` が最初にマッチした要素を返す
* `document.querySelectorAll` はマッチした要素すべてを NodeList で返す．`[].slice.call` を使えば Array に出来る．
* マッチする要素が無いときに jQuery が `[]` を返すのに対して DOM API は `null` を返すので，扱いには注意が必要．

> 注: `document.querySelector` と `document.querySelectorAll` はかなり **遅い** ので性能を求めるなら `getElementById` や `document.getElementsByClassName`, `document.getElementsByTagName` を使ってみたらいい．

- [1.0](#1.0) <a name='1.0'></a> セレクタ

  ```js
  // jQuery
  $('selector');

  // Native
  document.querySelectorAll('selector');
  ```

- [1.1](#1.1) <a name='1.1'></a> クラスセレクタ

  ```js
  // jQuery
  $('.class');

  // Native
  document.querySelectorAll('.class');
  
  // あるいは
  document.getElementsByClassName('class');

  ```

- [1.2](#1.2) <a name='1.2'></a> ID セレクタ

  ```js
  // jQuery
  $('#id');

  // Native
  document.querySelector('#id');

  // あるいは
  document.getElementById('id');
  ```

- [1.3](#1.3) <a name='1.3'></a> 属性セレクタ

  ```js
  // jQuery
  $('a[target=_blank]');

  // Native
  document.querySelectorAll('a[target=_blank]');
  ```

- [1.4](#1.4) <a name='1.4'></a> 何かの取得

  + ノードの取得

    ```js
    // jQuery
    $el.find('li');

    // Native
    el.querySelectorAll('li');
    ```

  + body の取得

    ```js
    // jQuery
    $('body');

    // Native
    document.body;
    ```

  + 属性値の取得

    ```js
    // jQuery
    $el.attr('foo');

    // Native
    e.getAttribute('foo');
    ```

  + data 属性値の取得

    ```js
    // jQuery
    $el.data('foo');

    // Native
    // getAttribute を使うか
    el.getAttribute('data-foo');
    // IE 11 以上だけなら `dataset` を使うことも出来る
    el.dataset['foo'];
    ```

- [1.5](#1.5) <a name='1.5'></a> 兄弟/前/後 要素

  + 兄弟要素

    ```js
    // jQuery
    $el.siblings();

    // Native
    [].filter.call(el.parentNode.children, function(child) {
      return child !== el;
    });
    ```

  + 前要素

    ```js
    // jQuery
    $el.prev();

    // Native
    el.previousElementSibling;

    ```

  + 後要素

    ```js
    // next
    $el.next();
    el.nextElementSibling;
    ```

- [1.6](#1.6) <a name='1.6'></a> 最近親

  親要素の方向にたどって最初にセレクタにマッチした要素を返す

  ```js
  // jQuery
  $el.closest(queryString);

  // Native
  function closest(el, selector) {
    const matchesSelector = el.matches || el.webkitMatchesSelector || el.mozMatchesSelector || el.msMatchesSelector;

    while (el) {
      if (matchesSelector.call(el, selector)) {
        return el;
      } else {
        el = el.parentElement;
      }
    }
    return null;
  }
  ```

- [1.7](#1.7) <a name='1.7'></a> Parents Until

  指定したセレクタにマッチするまで親要素を取得する．ただし，マッチした要素自体は含めない．

  ```js
  // jQuery
  $el.parentsUntil(selector, filter);

  // Native
  function parentsUntil(el, selector, filter) {
    const result = [];
    const matchesSelector = el.matches || el.webkitMatchesSelector || el.mozMatchesSelector || el.msMatchesSelector;

    // 親要素から探索
    el = el.parentElement;
    while (el && !matchesSelector.call(el, selector)) {
      if (!filter) {
        result.push(el);
      } else {
        if (matchesSelector.call(el, filter)) {
          result.push(el);
        }
      }
      el = el.parentElement;
    }
    return result;
  }
  ```

- [1.8](#1.8) <a name='1.8'></a> フォーム

  + Input/Textarea

    ```js
    // jQuery
    $('#my-input').val();

    // Native
    document.querySelector('#my-input').value;
    ```

  + `.radio` の中の currentTarget のインデックスの取得

    ```js
    // jQuery
    $(e.currentTarget).index('.radio');

    // Native
    [].indexOf.call(document.querySelectAll('.radio'), e.currentTarget);
    ```

- [1.9](#1.9) <a name='1.9'></a> iframe の内容

  `$('iframe').contents()` はその iframe の `contentDocument` を返す

  + iframe の内容

    ```js
    // jQuery
    $iframe.contents();

    // Native
    iframe.contentDocument;
    ```

  + iframe 内へのクエリー

    ```js
    // jQuery
    $iframe.contents().find('.css');

    // Native
    iframe.contentDocument.querySelectorAll('.css');
    ```

**[⬆ トップへ](#table-of-contents)**

## CSS & Style

- [2.1](#2.1) <a name='2.1'></a> CSS

  + style の取得

    ```js
    // jQuery
    $el.css("color");

    // Native
    // 注: 'auto' が設定されている場合，'auto' が返るバグがある
    const win = el.ownerDocument.defaultView;
    // 疑似要素ではないので null を明示
    win.getComputedStyle(el, null).color;
    ```

  + style の設定

    ```js
    // jQuery
    $el.css({ color: "#ff0011" });

    // Native
    el.style.color = '#ff0011';
    ```

  + 複数の値の取得と設定

    複数のスタイルを設定したいなら oui-dom-utils の [setStyles](https://github.com/oneuijs/oui-dom-utils/blob/master/src/index.js#L194) を参照下し．


  + class の追加

    ```js
    // jQuery
    $el.addClass(className);

    // Native
    el.classList.add(className);
    ```

  + class の削除

    ```js
    // jQuery
    $el.removeClass(className);

    // Native
    el.classList.remove(className);
    ```

  + class の有無

    ```js
    // jQuery
    $el.hasClass(className);

    // Native
    el.classList.contains(className);
    ```

  + class の付け外し

    ```js
    // jQuery
    $el.toggleClass(className);

    // Native
    el.classList.toggle(className);
    ```

- [2.2](#2.2) <a name='2.2'></a> 幅と高さ

  幅と高さはほぼおなじなので高さ ( Height ) で説明

  + Window の高さ

    ```js
    // window height
    $(window).height();
    // without scrollbar, behaves like jQuery
    window.document.documentElement.clientHeight;
    // with scrollbar
    window.innerHeight;
    ```

  + Document の高さ

    ```js
    // jQuery
    $(document).height();

    // Native
    document.documentElement.scrollHeight;
    ```

  + 要素の高さ

    ```js
    // jQuery
    $el.height();

    // Native
    function getHeight(el) {
      const styles = this.getComputedStyles(el);
      const height = el.offsetHeight;
      const borderTopWidth = parseFloat(styles.borderTopWidth);
      const borderBottomWidth = parseFloat(styles.borderBottomWidth);
      const paddingTop = parseFloat(styles.paddingTop);
      const paddingBottom = parseFloat(styles.paddingBottom);
      return height - borderBottomWidth - borderTopWidth - paddingTop - paddingBottom;
    }
    // 整数値で取得の場合 ( `border-box` では `height`．`content-box` では `height + padding + border` )
    el.clientHeight;
    // 実数値で取得の場合 ( `border-box` では `height`．`content-box` では `height + padding + border` )
    el.getBoundingClientRect().height;
    ```

- [2.3](#2.3) <a name='2.3'></a> 位置とオフセット

  + 位置

    ```js
    // jQuery
    $el.position();

    // Native
    { left: el.offsetLeft, top: el.offsetTop }
    ```

  + オフセット

    ```js
    // jQuery
    $el.offset();

    // Native
    function getOffset (el) {
      const box = el.getBoundingClientRect();

      return {
        top: box.top + window.pageYOffset - document.documentElement.clientTop,
        left: box.left + window.pageXOffset - document.documentElement.clientLeft
      }
    }
    ```

- [2.4](#2.4) <a name='2.4'></a> スクロールトップ

  ```js
  // jQuery
  $(window).scrollTop();

  // Native
  (document.documentElement && document.documentElement.scrollTop) || document.body.scrollTop;
  ```

**[⬆ トップへ](#table-of-contents)**

## DOM 操作

- [3.1](#3.1) <a name='3.1'></a> 削除
  ```js
  // jQuery
  $el.remove();

  // Native
  el.parentNode.removeChild(el);
  ```

- [3.2](#3.2) <a name='3.2'></a> テキスト

  + テキストの取得

    ```js
    // jQuery
    $el.text();

    // Native
    el.textContent;
    ```

  + テキストの設定

    ```js
    // jQuery
    $el.text(string);

    // Native
    el.textContent = string;
    ```

- [3.3](#3.3) <a name='3.3'></a> HTML

  + HTML の取得

    ```js
    // jQuery
    $el.html();

    // Native
    el.innerHTML;
    ```

  + HTML の設定

    ```js
    // jQuery
    $el.html(htmlString);

    // Native
    el.innerHTML = htmlString;
    ```

- [3.4](#3.4) <a name='3.4'></a> append

  親要素の最後尾に追加

  ```js
  // jQuery
  $el.append("<div id='container'>hello</div>");

  // Native
  let newEl = document.createElement('div');
  newEl.setAttribute('id', 'container');
  newEl.innerHTML = 'hello';
  el.appendChild(newEl);
  ```

- [3.5](#3.5) <a name='3.5'></a> prepend

  ```js
  // jQuery
  $el.prepend("<div id='container'>hello</div>");

  // Native
  let newEl = document.createElement('div');
  newEl.setAttribute('id', 'container');
  newEl.innerHTML = 'hello';
  el.insertBefore(newEl, el.firstChild);
  ```

- [3.6](#3.6) <a name='3.6'></a> insertBefore

  選択された要素の前にノードを追加

  ```js
  // jQuery
  $newEl.insertBefore(queryString);

  // Native
  newEl.insertBefore(document.querySelector(queryString));
  ```

- [3.7](#3.7) <a name='3.7'></a> insertAfter

  選択された要素の後ろにノードを追加

  ```js
  // jQuery
  $newEl.insertAfter(queryString);

  // Native
  function insertAfter(newEl, queryString) {
    const parent = document.querySelector(queryString).parentNode;

    if (parent.lastChild === newEl) {
      parent.appendChild(newEl);
    } else {
      parent.insertBefore(newEl, parent.nextSibling);
    }
  },
  ```

**[⬆ トップへ](#table-of-contents)**

## Ajax

 [fetch](https://github.com/camsong/fetch-ie8) や [fetch-jsonp](https://github.com/camsong/fetch-jsonp) に置き換える

**[⬆ トップへ](#table-of-contents)**

## イベント

名前空間やデリゲートまでサポートした代替実装は https://github.com/oneuijs/oui-dom-events を参照下し

- [5.1](#5.1) <a name='5.1'></a> 要素へのイベントのバインド

  ```js
  // jQuery
  $el.on(eventName, eventHandler);

  // Native
  el.addEventListener(eventName, eventHandler);
  ```

- [5.2](#5.2) <a name='5.2'></a> 要素からのイベントのアンバインド

  ```js
  // jQuery
  $el.off(eventName, eventHandler);

  // Native
  el.removeEventListener(eventName, eventHandler);
  ```

- [5.3](#5.3) <a name='5.3'></a> トリガー

  ```js
  // jQuery
  $(el).trigger('custom-event', {key1: 'data'});

  // Native
  if (window.CustomEvent) {
    const event = new CustomEvent('custom-event', {detail: {key1: 'data'}});
  } else {
    const event = document.createEvent('CustomEvent');
    event.initCustomEvent('custom-event', true, true, {key1: 'data'});
  }

  el.dispatchEvent(event);
  ```

**[⬆ トップへ](#table-of-contents)**

## ユーティリティ

- [6.1](#6.1) <a name='6.1'></a> isArray

  ```js
  // jQuery
  $.isArray(range);

  // Native
  Array.isArray(range);
  ```

- [6.2](#6.2) <a name='6.2'></a> trim

  ```js
  // jQuery
  $.trim(string);

  // Native
  String.trim(string);
  ```

- [6.3](#6.3) <a name='6.3'></a> Object Assign

  extend には object.assign の polyfill https://github.com/ljharb/object.assign を参照

  ```js
  // jQuery
  $.extend({}, defaultOpts, opts);

  // Native
  Object.assign({}, defaultOpts, opts);
  ```

- [6.4](#6.4) <a name='6.4'></a> contains

  ```js
  // jQuery
  $.contains(el, child);

  // Native
  el !== child && el.contains(child);
  ```

**[⬆ トップへ](#table-of-contents)**

## 翻訳

* [English](./README.md)
* [한국어](./README.ko-KR.md)
* [简体中文](./README.zh-CN.md)
* [Bahasa Melayu](README-my.md)

## ブラウザの対応状況

![Chrome](https://raw.github.com/alrra/browser-logos/master/chrome/chrome_48x48.png) | ![Firefox](https://raw.github.com/alrra/browser-logos/master/firefox/firefox_48x48.png) | ![IE](https://raw.github.com/alrra/browser-logos/master/internet-explorer/internet-explorer_48x48.png) | ![Opera](https://raw.github.com/alrra/browser-logos/master/opera/opera_48x48.png) | ![Safari](https://raw.github.com/alrra/browser-logos/master/safari/safari_48x48.png)
--- | --- | --- | --- | --- |
Latest ✔ | Latest ✔ | 10+ ✔ | Latest ✔ | 6.1+ ✔ |

# License

MIT
