# atcoder-oj-env

AtCoder CLI (acc) と Online Judge Tools (oj) を使って、問題取得・テスト・提出を行うための最小セットアップ手順です。

Python パッケージ管理は uv 前提にしています。競技用ライブラリと oj は [pyproject.toml](pyproject.toml) で管理します。

## 前提条件

`acc` は Node.js、Python 側は Python 3.13 + uv を使います。

### macOS

```bash
brew install node uv
```

### Ubuntu / Linux

```bash
sudo apt update
sudo apt install nodejs npm
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### Windows

- Node.js: 公式インストーラで導入
- uv: `winget install --id AstralSoftware.uv` で導入

必要なら uv で Python 3.13 も入れられます。

```powershell
uv python install 3.13
```

### インストール確認

```bash
node --version
npm --version
uv --version
```

## uv 環境の作成

このリポジトリには Python 3.13 以上を前提とした依存定義を入れています。

```bash
uv sync
```

これで `.venv` が作成され、次のパッケージが入ります。

- `numpy==2.3.2`
- `online-judge-tools==11.5.1`
- `pandas==2.3.1`
- `pyyaml>=6.0.3`
- `ruff>=0.15.4`
- `setuptools>=82.0.1`
- `sortedcontainers==2.4.0`

環境確認は次でできます。

```bash
uv run python --version
uv run python -c "import numpy, pandas, yaml, sortedcontainers; print('ok')"
uv run ruff --version
uv run oj -h
```

## acc のインストール

```bash
npm install -g atcoder-cli
acc -h
```

## acc の設定

ここは重要です。

`acc` の設定はこのリポジトリの中ではなく、`acc config-dir` が指すユーザーごとの設定ディレクトリに保存されます。

つまり、新しい Mac や別の PC を買ったときは、リポジトリを clone するだけでは不十分です。`acc` のテンプレートと設定は、そのマシンで再作成する必要があります。

このリポジトリの Git 管理外にあるものは、少なくとも次です。

- `acc config-dir` 配下の `python/main.py`
- `acc config-dir` 配下の `python/template.json`
- `acc config-dir` 配下の `python/generate_main.ps1` または `python/generate_main.sh`
- `acc config default-template python` で設定される `default-template`
- `acc config oj-path ...` で設定される `oj-path`
- `acc login` や手動 cookie 設定で保存されるログイン情報

つまり、新しいマシンでは `uv sync` だけでは足りず、`acc` のテンプレート配置、`default-template`、`oj-path`、ログイン状態を作り直す必要があります。

### 1. 設定ディレクトリを確認する

```bash
acc config-dir
```

ここで表示されたディレクトリ配下に、テンプレートを作ります。

### 2. `python` テンプレートを作る

`acc config-dir` 配下に `python` ディレクトリを作り、少なくとも次の 2 ファイルを置きます。

- `python/main.py`
- `python/template.json`

1 行目へ問題 URL を自動で入れたい場合は、追加で補助スクリプトも置きます。

- Windows: `python/generate_main.ps1`
- macOS / Linux: `python/generate_main.sh`

ポイントは次です。

- `python/main.py` だけ置いてもテンプレートとしては認識されません
- `python/template.json` があって初めて、`acc` はそのディレクトリをテンプレートとして扱います

### 3. `python/main.py` を作る

これは各問題ディレクトリにコピーされる Python のひな形です。

例:

```python
# URL will be injected by acc template.
from collections import Counter, defaultdict, deque
from heapq import heappop, heappush
from bisect import bisect_left, bisect_right
from math import comb, factorial, gcd, lcm, perm
from itertools import accumulate, combinations, permutations, product
from functools import lru_cache
import operator
from string import ascii_lowercase, ascii_uppercase, digits

MOD = 998244353


def II() -> int:
		return int(input())


def LI() -> list[str]:
		return list(input())


def LMI() -> list[int]:
		return list(map(int, input().split()))


def LMS() -> list[str]:
		return list(map(str, input().split()))


def LLMI(x: int) -> list[list[int]]:
		return [list(map(int, input().split())) for _ in range(x)]


def LLMS(x: int) -> list[list[str]]:
		return [list(input()) for _ in range(x)]


def execute() -> None:
		pass


if __name__ == "__main__":
		T = 1
		for _ in range(T):
				execute()
```

### 4. `python/template.json` を作る

このファイルで、`acc` に「どのファイルを生成するか」を教えます。

Windows の場合:

```json
{
	"task": {
		"program": ["main.py"],
		"submit": "main.py",
		"cmd": "powershell -ExecutionPolicy Bypass -File \"%TEMPLATE_DIR%\\generate_main.ps1\""
	}
}
```

macOS / Linux の場合:

```json
{
	"task": {
		"program": ["main.py"],
		"submit": "main.py",
		"cmd": "sh \"$TEMPLATE_DIR/generate_main.sh\""
	}
}
```

`cmd` が不要なら省略できますが、その場合は問題 URL の自動挿入はされません。

### 5. 問題 URL 自動挿入スクリプトを作る

Windows の場合は `python/generate_main.ps1` を置きます。

```powershell
$programPath = Join-Path $env:TASK_DIR "main.py"
$contestId = $env:CONTEST_ID.ToLower()
$taskId = $env:TASK_ID.ToLower()
if ($taskId.StartsWith("$contestId_")) {
		$taskSlug = $taskId
} else {
		$taskSlug = "$contestId`_$taskId"
}
$url = "https://atcoder.jp/contests/$contestId/tasks/$taskSlug"
$content = Get-Content -LiteralPath $programPath
$content[0] = "# $url"
$encoding = New-Object System.Text.UTF8Encoding($false)
[System.IO.File]::WriteAllLines($programPath, $content, $encoding)
```

macOS / Linux の場合は `python/generate_main.sh` を置きます。

```sh
#!/bin/sh
program_path="$TASK_DIR/main.py"
contest_id=$(printf '%s' "$CONTEST_ID" | tr '[:upper:]' '[:lower:]')
task_id=$(printf '%s' "$TASK_ID" | tr '[:upper:]' '[:lower:]')

case "$task_id" in
	${contest_id}_*) task_slug="$task_id" ;;
	*) task_slug="${contest_id}_${task_id}" ;;
esac

url="https://atcoder.jp/contests/${contest_id}/tasks/${task_slug}"

{
	printf '# %s\n' "$url"
	tail -n +2 "$program_path"
} > "$program_path.tmp"
mv "$program_path.tmp" "$program_path"
```

### 6. デフォルトテンプレートを設定する

テンプレートを作っただけでは、自動では使われません。`default-template` を設定して初めて `acc new` で毎回使われます。

```bash
acc config default-template python
```

確認:

```bash
acc config default-template
```

`python` と表示されればOKです。

### 7. `oj-path` を設定する

`acc` が `oj` を呼び出すときの実行ファイルパスも、マシンごとに設定が必要です。

Windows 例:

```powershell
acc config oj-path C:\Users\user\Desktop\atcoder-oj-env\.venv\Scripts\oj.exe
```

macOS / Linux 例:

```bash
acc config oj-path "$(pwd)/.venv/bin/oj"
```

確認:

```bash
acc check-oj
```

### 8. 何を直せば何が変わるか

- `python/main.py`
: 各問題にコピーされるコードひな形そのもの
- `python/template.json`
: どのファイルを生成するか、何を submit 対象にするか
- `python/generate_main.ps1` / `python/generate_main.sh`
: 1 行目の URL 自動挿入の挙動

普段のテンプレートを変えたいだけなら、まず `acc config-dir` 配下の `python/main.py` を編集すれば足ります。

## oj のインストール

`oj` は `pyproject.toml` に含めているため、追加インストールは不要です。

```bash
uv sync
uv run oj -h
```

Python 3.13 では `distutils` 互換のために `setuptools` も一緒に入れています。

グローバルコマンドとして使いたい場合だけ、任意で tool インストールにしてください。

```bash
uv tool install online-judge-tools
```

普段は `uv run oj ...` で十分です。

## 連携確認

```bash
acc check-oj
```

`available` と表示されれば連携できています。

## ログイン

```bash
acc login
uv run oj login https://atcoder.jp
```

`acc login` がうまく通らない場合は、AtCoder 側の認証変更の影響を受けていることがあります。その場合はブラウザから `REVEL_SESSION` を取得して、`session.json` に手動設定する方法が確実です。

```bash
acc config-dir
```

上のコマンドで表示された設定ディレクトリ内の `session.json` を、次の形式で保存します。

```json
{"REVEL_SESSION": "取得した値"}
```

## 基本的な使い方

### コンテストを展開

```bash
acc new abc449
```

問題選択画面では次の操作が使えます。

- `a` で全問題選択
- `Enter` で展開

`acc config-dir` 配下に `python` テンプレート一式を作成し、`acc config default-template python` を設定済みなら、各問題ディレクトリに `main.py` と `tests` が作られます。

生成例:

```text
abc449/
├── a/
│   ├── main.py
│   └── tests/
├── b/
│   ├── main.py
│   └── tests/
└── contest.acc.json
```

`main.py` の 1 行目には問題 URL が自動で入ります。

### テスト

```bash
cd abc449/a
uv run oj t -c "uv run python main.py"
```

### 提出

```bash
cd abc449/a
acc submit main.py
```

### テンプレートを直す

テンプレート修正は、上の `acc の設定` 章にある `acc config-dir` 配下の `python` テンプレートを編集します。

特に、普段のひな形を変えたいだけなら `python/main.py` を編集すれば足ります。

## VS Code のタスクとショートカット設定

VS Code で test / submit をショートカット実行したい場合は、ワークスペースにタスクを置き、ユーザー設定にキーバインドを追加します。

ここで管理場所が分かれます。

- `.vscode/tasks.json` はこのリポジトリ内なので Git 管理できます
- `keybindings.json` は VS Code のユーザー設定なので Git 管理外です

新しい Mac や別 PC では、`keybindings.json` 側は手動で再設定が必要です。

### 1. タスク設定

`.vscode/tasks.json` を作成して、次を入れます。

```json
{
	"version": "2.0.0",
	"tasks": [
		{
			"label": "AtCoder: Test Current File",
			"type": "shell",
			"command": "uv run oj t -c \"uv run python ${fileBasename}\"",
			"options": {
				"cwd": "${fileDirname}"
			},
			"group": {
				"kind": "build",
				"isDefault": true
			},
			"presentation": {
				"reveal": "always",
				"panel": "shared",
				"clear": true
			},
			"problemMatcher": []
		},
		{
			"label": "AtCoder: Submit Current File",
			"type": "shell",
			"command": "acc submit ${fileBasename}",
			"options": {
				"cwd": "${fileDirname}"
			},
			"presentation": {
				"reveal": "always",
				"panel": "shared",
				"clear": true
			},
			"problemMatcher": []
		}
	]
}
```

### 2. ショートカット設定

VS Code の `keybindings.json` に次を追加します。

Windows なら通常は `C:\Users\user\AppData\Roaming\Code\User\keybindings.json` です。

macOS なら通常は `~/Library/Application Support/Code/User/keybindings.json` です。

```json
[
	{
		"key": "ctrl+alt+t",
		"command": "workbench.action.tasks.runTask",
		"args": "AtCoder: Test Current File",
		"when": "editorTextFocus"
	},
	{
		"key": "ctrl+alt+s",
		"command": "workbench.action.tasks.runTask",
		"args": "AtCoder: Submit Current File",
		"when": "editorTextFocus"
	}
]
```

### 3. 使い方

問題ディレクトリの `main.py` を開いた状態で使います。

- `Ctrl+Alt+T` で test
- `Ctrl+Alt+S` で submit

## よくあるエラー

### `abc449 file/directory already exists.`

同名フォルダが既に存在しています。不要なら削除して再実行してください。

```powershell
rmdir /s abc449
acc new abc449
```

### `spawn ... oj.exe ENOENT`

`acc` が内部で呼び出す `oj.exe` が見つかっていません。まず `oj` をインストールし、PATH が通っているか確認してください。

特に、過去の別環境の `oj.exe` を `acc` が覚えているケースが多いです。現在の設定確認と修正は次でできます。

```bash
acc config oj-path
acc config oj-path C:\Users\user\Desktop\atcoder-oj-env\.venv\Scripts\oj.exe
acc check-oj
```

確認

```bash
uv run oj -h
acc check-oj
```