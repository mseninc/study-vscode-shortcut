---
marp: true
---
<!-- header: VS Code で高める業務効率 -->
<!-- footer: kenzauros / 2020.03.07 -->
<!-- theme: default -->
<!-- class: invert -->
<!-- size: 16:9 -->
<!-- page_number: true -->
<!-- paginate: true -->

# VS Code で高める業務効率

```js
{
  date: '2020-03-07',
  author: 'kenzauros',
}
```

---

### 目的

VS Code を使いこなして仕事の効率を高めることを目的とします。

- 編集系
- 選択系
- マルチカーソル系

### ソース

題材となるファイルをダウンロードするため、 git clone し、フォルダを VS Code で開いてください。

```
git clone https://github.com/mseninc/z03-vscode-shortcut.git
```

---

## 編集系

---

### 課題1

- 7 行目の `command` あたりにカーソルをおいたまま、その行の下に空行を追加してください。

---

### 課題2

- 7 行目の `command` あたりにカーソルをおいたまま、その行の上に空行を追加してください。

---

### 課題3

- 22 行目の `mode` の行を 21 行目の `remote_src` の行と入れ替えてください。

---

### 課題4

- 10 行目をコピーして 15 行目の下にペーストしてください。

---

## 選択系

---

### 課題5

- 19 行目の `/etc/zabbix` の部分を選択してください。

---

### 課題6

- 26 行目の `lineinfile` の部分を先ほどとは違う方法で選択してください。

---

### 課題7

- 27 行目の全体を選択してください。

---

## マルチカーソル系

---

### 課題8

- 27 行目から 30 行目の先頭をまとめて選択して `#` を挿入してください。

---

### 課題9

- 26 行目の `lineinfile` を選択したまま 34 行目の `lineinfile` も選択してください。

---

### 課題10

- ファイル内のすべての `- name: ` を選択し、直後に `Zabbix ` と追加してください。
- 例
    - before: `- name: yum キャッシュの削除`
    - after: `- name: Zabbix yum キャッシュの削除`

---

## 発展系

---

### 課題11

- 63 行目の `redhat_yum_conf` を `RedhatYumConf` に変更してください。

---

### 課題12

- 63 行目の `RedhatYumConf` を `redhat-yum-conf` に変更してください。
