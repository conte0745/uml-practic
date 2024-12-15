# スクリプト

## テンプレートの作成

### ファイルの作成

``` vim
vim ~/.gitmessage.txt
```

### ./gitmessage の内容

```txt
[branchname]
```

## ブランチ名の置換Script

### hooks/commit-msg

```sh
#!/bin/sh

# 引数としてファイル名を受け取る
message_file=$1

# メッセージファイルをUTF-8エンコーディングで読み込む
message=$(cat "$message_file")

# 現在のGitブランチ名を取得（エラー処理を追加）
current_branch=$(git rev-parse --abbrev-ref HEAD 2>/dev/null)

# スラッシュがあるか確認
if echo "$current_branch" | grep -q '/'; then
    # スラッシュで区切って2番目の部分を取得
    current_branch=$(echo "$current_branch" | cut -d'/' -f2)
fi

# Gitのブランチが取得できなかった場合、エラーメッセージを出力
if [ $? -ne 0 ]; then
    echo "Error: Git is not initialized or not available in this directory."
    exit 1
fi

# メッセージ内の [branchname] を現在のブランチ名に置換
new_message=$(echo "$message" | sed "s/\[branchname\]/$current_branch/g")

# 新しいメッセージをファイルに書き込む
echo "$new_message" > "$message_file"
```

## シークレット検知

### hooks/pre-commit 1

```sh
#!/bin/sh
# 除外する相対パスのファイルやディレクトリをリスト化
EXCLUDE_PATHS=("git-template/git-readme.md")

# 除外コマンドを組み立てる
EXCLUDE_CMD=""
for path in "${EXCLUDE_PATHS[@]}"; do
    EXCLUDE_CMD="$EXCLUDE_CMD -v $path "
done

# ステージされた変更を取得し、指定されたファイルを除外しつつ、KEY と AKIA を含む行を検索
GREP_RESULT=$(git diff --cached --name-only | eval "grep $EXCLUDE_CMD" | xargs -I {} git diff --cached {} | grep KEY | grep AKIA)

# 結果が見つかった場合
if [ -n "$GREP_RESULT" ]; then
    echo 'AWS_ACCESS_KEY might be in this index. Please check with git diff --cached'
    echo "$GREP_RESULT"
    exit 1
fi
```

## コンフリクト記号検知

### hooks/pre-commit 2

```sh

# 除外したいファイルやディレクトリをリスト化
EXCLUDE_FILES=("git-template/git-readme.md")

# Simple check for merge conflicts
conflicts=$(git diff --cached --name-only -G"<<<<<|=====|>>>>>" )

# 除外ファイルを適用
for file in "${EXCLUDE_FILES[@]}"; do
    conflicts=$(echo "$conflicts" | grep -v "$file")
done

# If conflicts are found
if [ -n "$conflicts" ]; then
    echo "未解決のコンフリクトを解消してください:"

    # Loop through each conflicted file
    for conflict in $conflicts; do
        echo "$conflict"
    done
    exit 1
fi
```

## 全てのリポジトリに適用する

[globalなgit-hooksを設定して、すべてのリポジトリで共有のhooksを使う](https://qiita.com/ik-fib/items/55edad2e5f5f06b3ddd1)

## 個別のリポジトリは無視してグローバルのhooksのみ処理

```sh
cp -r .git/hooks ~/.config/git/hooks
```

```gitconfig
[core]
    hooksPath = ~/.config/git/hooks
```
