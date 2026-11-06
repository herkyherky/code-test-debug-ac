# 可审阅的源码目录导入方案（尚未执行）

完整源码已在 Release 的 AC-0.8.0-source.zip 内公开。将其展开为仓库中逐文件可浏览的源码，需要一次性导入。

当前 GitHub 连接对新仓库的内容写入返回 403；网页上传这批原生构建材料时提示文件过大。下面的方案使用 GitHub 官方 Actions 运行器和该仓库自带的临时 GITHUB_TOKEN，不需要个人访问令牌，不扩大 GitHub 连接的授权范围。

该方案会新建一个仅能由仓库所有者手动运行的工作流，授予该次任务 contents: write，下载指定版本的源码附件、检查固定 SHA-256、解压并把文件推送到 main。发现 app 已存在会停止，不执行下载源码中的应用代码。它会产生源码提交，工作流文件也会留在仓库内，属于安全敏感能力，需所有者明确同意后才会添加或运行。

自动审批已拒绝未经另行授权创建此工作流；本文件只是方案文本，不是有效工作流，不会执行。

## 待批准的工作流

```yaml
name: Import verified AC v0.8.0 source

on:
  workflow_dispatch:

permissions:
  contents: write

jobs:
  import-source:
    if: github.actor == github.repository_owner
    runs-on: ubuntu-latest
    timeout-minutes: 10
    env:
      GH_TOKEN: ${{ github.token }}
      AC_ARCHIVE_SHA256: ea8311603ac418ccb217886ef26a3162a557a2f0cfdfb9fee6de32a8cae3070d
    steps:
      - name: Import checksum-verified release source into main
        shell: bash
        run: |
          set -euo pipefail
          gh auth setup-git
          AC_IMPORT_DIR="$RUNNER_TEMP/ac-public-import"
          AC_RELEASE_DIR="$RUNNER_TEMP/ac-release-source"
          gh repo clone "$GITHUB_REPOSITORY" "$AC_IMPORT_DIR" -- --branch main
          cd "$AC_IMPORT_DIR"
          if test -f app/src/main/AndroidManifest.xml; then
            echo "Source is already imported; refusing to overwrite an existing project."
            exit 1
          fi
          mkdir -p "$AC_RELEASE_DIR"
          gh release download v0.8.0 --repo "$GITHUB_REPOSITORY" --pattern AC-0.8.0-source.zip --dir "$AC_RELEASE_DIR"
          printf '%s  %s\n' "$AC_ARCHIVE_SHA256" "$AC_RELEASE_DIR/AC-0.8.0-source.zip" | sha256sum --check --status
          export AC_RELEASE_DIR
          python3 - <<'PYCODE'
          from pathlib import Path, PurePosixPath
          import os, shutil, stat, zipfile
          archive = Path(os.environ['AC_RELEASE_DIR']) / 'AC-0.8.0-source.zip'
          extracted = archive.parent / 'extracted'
          with zipfile.ZipFile(archive) as z:
              for info in z.infolist():
                  name = PurePosixPath(info.filename)
                  if name.is_absolute() or '..' in name.parts or not name.parts or name.parts[0] != 'AC-0.8.0':
                      raise RuntimeError('Invalid archive path')
                  if stat.S_ISLNK(info.external_attr >> 16):
                      raise RuntimeError('Symlinks are not accepted')
              z.extractall(extracted)
          root = extracted / 'AC-0.8.0'
          if not (root / 'app/src/main/AndroidManifest.xml').is_file():
              raise RuntimeError('Source manifest missing')
          shutil.copytree(root, Path.cwd(), dirs_exist_ok=True)
          for name in ['build.sh', 'prebuilt/pocket-runner-arm64', 'prebuilt/gdb-18.1-minimal-arm64']:
              Path(name).chmod(0o755)
          PYCODE
          git config user.name 'github-actions[bot]'
          git config user.email '41898282+github-actions[bot]@users.noreply.github.com'
          git add -A
          git commit -m 'Open-source AC v0.8.0: native C++, GDB, sample tests and optional OJ'
          git push origin HEAD:main
          echo "Imported source commit: $(git rev-parse HEAD)"
```
