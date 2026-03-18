# Quick Contest Guide

コンテスト前に見る用の最短手順です。

## 0. 最初だけ

```powershell
uv sync
acc check-oj
```

`available` と出ればOKです。

## 1. コンテストを作る

例: ABC449

```powershell
acc new abc449
```

操作:

- `a` で全問題選択
- `Enter` で作成

作成後の形:

```text
abc449/
├── a/
│   ├── main.py
│   └── tests/
├── b/
│   ├── main.py
│   └── tests/
└── ...
```

## 2. 解く

A 問題ならここを開く。

- [abc449/a/main.py](abc449/a/main.py)

1 行目には問題 URL が自動で入っています。

## 3. テストする

```powershell
cd abc449\a
uv run oj t -c "uv run python main.py"
```

VS Code では今開いている `main.py` に対して次で実行できます。

- `Ctrl+Alt+T` で test

## 4. 提出する

```powershell
acc submit main.py
```

VS Code では今開いている `main.py` に対して次で実行できます。

- `Ctrl+Alt+S` で submit

## 5. 次の問題へ

```powershell
cd ..\b
uv run oj t -c "uv run python main.py"
acc submit main.py
```

## ハマったとき

`oj.exe ENOENT` が出たら:

```powershell
acc config oj-path C:\Users\user\Desktop\atcoder-oj-env\.venv\Scripts\oj.exe
acc check-oj
```

`abc449 already exists` が出たら:

```powershell
acc new abc449 --force --choice all
```