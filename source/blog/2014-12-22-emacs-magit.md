---
title: "magitを使い始めて、evilといい感じに組み合わせた話"
date: "2014-12-22 22:18:45"
---

この記事は[Emacs Advent Calendar](http://qiita.com/advent-calendar/2014/emacs)の22日目です。

若干今更感のある話ですが、最近[Magit](http://magit.github.io)を使い始めました。
ところで僕はevilを使っていて、evilとmagitを同時に使うとデフォルトだとかなり微妙な感じになるのでevilとmagitを一緒に使ってもいい感じになるようにいい感じにカスタマイズしたので紹介します。

## 一応、Magitとは

emacs上で動作するgitクライアントです。普段gitを使ってするような操作は多分大体magitで実現できます。

## magitの設定

以下僕の設定ファイルからコピ&ペです。

```
(defadvice magit-status (around magit-fullscreen activate)
  (window-configuration-to-register :magit-fullscreen)
  ad-do-it
  (delete-other-windows)
  )

(defun magit-quit-session ()
  "Restores the previous window configuration and kills the magit buffer"
  (interactive)
  (kill-buffer)
  (jump-to-register :magit-fullscreen))

(defun magit-toggle-whitespace ()
  (interactive)
  (if (member "-w" magit-diff-options)
      (magit-dont-ignore-whitespace)
    (magit-ignore-whitespace)))

(defun magit-ignore-whitespace ()
  (interactive)
  (add-to-list 'magit-diff-options "-w")
  (magit-refresh))

(defun magit-dont-ignore-whitespace ()
  (interactive)
  (setq magit-diff-options (remove "-w" magit-diff-options))
  (magit-refresh))

;;;;;;;;;;;;;;;;;;;
;; evil key bindings
;;;;;;;;;;;;;;;;;;;

(evil-set-initial-state 'magit-log-edit-mode 'insert)
(evil-set-initial-state 'git-commit-mode 'insert)
(evil-set-initial-state 'magit-commit-mode 'motion)
(evil-set-initial-state 'magit-status-mode 'motion)
(evil-set-initial-state 'magit-log-mode 'motion)
(evil-set-initial-state 'magit-wassup-mode 'motion)
(evil-set-initial-state 'magit-mode 'motion)
(evil-set-initial-state 'git-rebase-mode 'motion)

(evil-define-key 'motion git-rebase-mode-map
  "c" 'git-rebase-pick
  "r" 'git-rebase-reword
  "s" 'git-rebase-squash
  "e" 'git-rebase-edit
  "f" 'git-rebase-fixup
  "y" 'git-rebase-insert
  "d" 'git-rebase-kill-line
  "u" 'git-rebase-undo
  "x" 'git-rebase-exec
  (kbd "<return>") 'git-rebase-show-commit
  "\M-n" 'git-rebase-move-line-down
  "\M-p" 'git-rebase-move-line-up)

(evil-define-key 'motion magit-commit-mode-map
  "\C-c\C-b" 'magit-show-commit-backward
  "\C-c\C-f" 'magit-show-commit-forward)

(evil-define-key 'motion magit-status-mode-map
  "\C-f" 'evil-scroll-page-down
  "\C-b" 'evil-scroll-page-up
  "." 'magit-mark-item
  "=" 'magit-diff-with-mark
  "C" 'magit-add-log
  "I" 'magit-ignore-item-locally
  "S" 'magit-stage-all
  "U" 'magit-unstage-all
  "W" 'magit-toggle-whitespace
  "X" 'magit-reset-working-tree
  "d" 'magit-discard-item
  "i" 'magit-ignore-item
  "s" 'magit-stage-item
  "u" 'magit-unstage-item
  "z" 'magit-key-mode-popup-stashing)

(evil-define-key 'motion magit-log-mode-map
  "." 'magit-mark-item
  "=" 'magit-diff-with-mark
  "e" 'magit-log-show-more-entries)

(evil-define-key 'motion magit-wazzup-mode-map
  "." 'magit-mark-item
  "=" 'magit-diff-with-mark
  "i" 'magit-ignore-item)

(evil-set-initial-state 'magit-branch-manager-mode 'motion)
(evil-define-key 'motion magit-branch-manager-mode-map
  "a" 'magit-add-remote
  "c" 'magit-rename-item
  "d" 'magit-discard-item
  "o" 'magit-create-branch
  "v" 'magit-show-branches
  "T" 'magit-change-what-branch-tracks)

(evil-define-key 'motion magit-mode-map
  "\M-1" 'magit-show-level-1-all
  "\M-2" 'magit-show-level-2-all
  "\M-3" 'magit-show-level-3-all
  "\M-4" 'magit-show-level-4-all
  "\M-H" 'magit-show-only-files-all
  "\M-S" 'magit-show-level-4-all
  "\M-h" 'magit-show-only-files
  "\M-s" 'magit-show-level-4
  "!" 'magit-key-mode-popup-running
  "$" 'magit-process
  "+" 'magit-diff-larger-hunks
  "-" 'magit-diff-smaller-hunks
  "=" 'magit-diff-default-hunks
  "/" 'evil-search-forward
  ":" 'evil-ex
  ";" 'magit-git-command
  "?" 'evil-search-backward
  "<" 'magit-key-mode-popup-stashing
  "A" 'magit-cherry-pick-item
  "B" 'magit-key-mode-popup-bisecting
  "D" 'magit-revert-item
  "E" 'magit-ediff
  "F" 'magit-key-mode-popup-pulling
  "G" 'evil-goto-line
  "H" 'magit-rebase-step
  "J" 'magit-key-mode-popup-apply-mailbox
  "K" 'magit-key-mode-popup-dispatch
  "L" 'magit-add-change-log-entry
  "M" 'magit-key-mode-popup-remoting
  "N" 'evil-search-previous
  "P" 'magit-key-mode-popup-pushing
  "Q" 'magit-quit-session
  "R" 'magit-refresh-all
  "S" 'magit-stage-all
  "U" 'magit-unstage-all
  "W" 'magit-diff-working-tree
  "X" 'magit-reset-working-tree
  "Y" 'magit-interactive-rebase
  "Z" 'magit-key-mode-popup-stashing
  "a" 'magit-apply-item
  "b" 'magit-key-mode-popup-branching
  "c" 'magit-key-mode-popup-committing
  "e" 'magit-diff
  "f" 'magit-key-mode-popup-fetching
  "g?" 'magit-describe-item
  "g$" 'evil-end-of-visual-line
  "g0" 'evil-beginning-of-visual-line
  "gE" 'evil-backward-WORD-end
  "g^" 'evil-first-non-blank-of-visual-line
  "g_" 'evil-last-non-blank
  "gd" 'evil-goto-definition
  "ge" 'evil-backward-word-end
  "gg" 'evil-goto-first-line
  "gj" 'evil-next-visual-line
  "gk" 'evil-previous-visual-line
  "gm" 'evil-middle-of-visual-line
  "h" 'magit-key-mode-popup-rewriting
  "j" 'magit-goto-next-section
  "k" 'magit-goto-previous-section
  "l" 'magit-key-mode-popup-logging
  "m" 'magit-key-mode-popup-merging
  "n" 'evil-search-next
  "o" 'magit-key-mode-popup-submodule
  "p" 'magit-cherry
  "q" 'magit-mode-quit-window
  "r" 'magit-refresh
  "t" 'magit-key-mode-popup-tagging
  "v" 'magit-revert-item
  "w" 'magit-wazzup
  "x" 'magit-reset-head
  "y" 'magit-copy-item-as-kill
  ;z  position current line
  " " 'magit-show-item-or-scroll-up
  "\d" 'magit-show-item-or-scroll-down
  "\t" 'magit-visit-item
  (kbd "<return>")   'magit-toggle-section
  (kbd "C-<return>") 'magit-dired-jump
  (kbd "<backtab>")  'magit-expand-collapse-section
  (kbd "C-x 4 a")    'magit-add-change-log-entry-other-window
  (kbd "\M-d") 'magit-copy-item-as-kill)
```

## 解説

### キーバインド

この設定、もとはどこかからパクってきたものを改変してるんですが、ネタ元を忘れてしまいました。
まあ見てのとおり、各magitモードでもvimっぽいキーバインドで動作するように変更しています。

### magit-status

僕が使うmagitのだいたいの機能(status, log, rebase, commit, push)へのエントリポイントは`M-x magit-status`で統一しています。何故かというと、僕はmagit起動時に全画面になって欲しいので次の設定を`init.el`に追加して`magit-status`が常に全画面で起動するようにしているからです。

```
(defadvice magit-status (around magit-fullscreen activate)
  (window-configuration-to-register :magit-fullscreen)
  ad-do-it
  (delete-other-windows))
```

ついでに、magit起動前の状態にemacsをリストアする設定。ぼくはこれを`Q`に割り当ててます。

```
(defun magit-quit-session ()
  (interactive)
  (kill-buffer)
  (jump-to-register :magit-fullscreen))
```

これだけの話です。evilユーザーは残念ながらそんなに多くないだろうと想像するので、今回紹介した設定が役に立つ人は少ないかもしれませんがこれを機にmagitと、できればevilを使い始めてみてはいかがでしょうか。

余談ですがmagitで一番便利だと感じてる機能はinteractive rebaseです。次点で`magit-blame-mode`です。他の機能は僕の使う範囲では正直tigとかgitコマンドとかと比べて明確なメリットはないです。が、全部の操作がemacsで完結するのはすごく気分がいいので僕はgit操作はmagitに統一しました。

最近zshからeshellに移行したという話もあるんですがそれはまた別の記事で。
すべての作業がemacsで完結する環境を目指してます。
