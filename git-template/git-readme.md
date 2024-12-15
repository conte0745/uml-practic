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

### hooks/pre-commit

```sh

#!/bin/sh

GREP_RESULT=$(git diff --cached | grep KEY | grep AKIA)

if [ -n "${GREP_RESULT}" ]; then
    echo 'AWS_ACCESS_KEY might be in this index. Please check with git diff --cached'
    echo "${GREP_RESULT}"
    exit 1
fi
```