
## Adjusted Prettier Opinionated Code Formatter

See original prettier source code (v2.8.8) - https://github.com/prettier/prettier

> ---
> ### This package provides only formatting for Vue files template section! 
> ---

#### PRETTIER CONFIG

This package expects specific configuration to work as intended (if different config is used, the results are unknown). Put following configuration into `.prettierrc` file in the root of your project:

```json
{
    "tabWidth": 4,
    "printWidth": 120,
    "semi": false,
    "singleQuote": true,
    "quoteProps": "consistent",
    "bracketSameLine": true,
    "htmlWhitespaceSensitivity": "ignore",
    "vueIndentScriptAndStyle": true,
    "singleAttributePerLine": true
}
```

#### HOW TO FORMAT A FILE

Run formatter from CLI with following command:

```sh
npx zero-prettier --parser vue --write [pathToFile]
```

#### INSTALLATION

You can install package as dev dependency for your project with following command:

```sh
npm install -D zero-prettier
```

#### GIT POST-COMMIT HOOK

If you feel confident in formatter, you can use it in git post-commit hook to automatically format only changed Vue files in the last commit. If some vue files will be formatted, then this script will create new commit for it. Create file `.git/hooks/post-commit` with following content:

```sh
#!/bin/bash

# Get the list of .vue files changed in the last commit
LAST_COMMIT_FILES=$(git diff-tree --no-commit-id --name-only -r HEAD | grep ".vue$")

# Check if there are any Vue files in the last commit
if [[ -z "$LAST_COMMIT_FILES" ]]; then
    echo "No Vue files were changed in the last commit. Skipping 'zero-prettier' formatting."
    exit 0
fi

echo "Formatting last commit's Vue files with 'zero-prettier'..."
echo "$LAST_COMMIT_FILES"

# Run `zero-prettier` on each changed Vue file
for file in $LAST_COMMIT_FILES; do
    # Perform formatting twice because attributes are formatted after children, thus children might not have empty line before parent opening tag end
    for i in {1..2}; do
        npx zero-prettier --parser vue --write --ignore-unknown "$file"

        if [[ $? -ne 0 ]]; then
            echo "'zero-prettier' failed to format $file. Aborting commit."
            exit 1
        fi
    done
done

# Check if `zero-prettier` made changes
git diff --quiet --exit-code $LAST_COMMIT_FILES
if [[ $? -eq 0 ]]; then
    echo "No formatting changes detected. Skipping creating new commit with formatted files."
    exit 0
fi

# Stage and commit formatted files
git add $LAST_COMMIT_FILES
git commit -m "(hook) FORMATTING: formatted last commit's Vue files with 'zero-prettier' code formatter"

echo "'zero-prettier' formatting applied and committed!"
exit 0
```
