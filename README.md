前情提要可參考：[如何用 GitHub Workflow 實現自動 Freeze Main Branch](https://blog.hubertyang.com/post/auto-freeze-branch-via-github-workflow/)

# 1️⃣ Merge Freeze Blocker

**用途 / Purpose：**  
防止在「merge freeze」期間，PR 被誤合併進 main，確保 release 期間程式碼穩定。  
Prevent PRs from being merged into `main` during a "merge freeze" period, ensuring code stability during releases.

**運作方式 / How it works：**

- 只要 PR 上有 `merge-freeze` 這個 label，這個 workflow 會讓 PR 的檢查狀態變成 failed，強制不能 merge。  
  If a PR has the `merge-freeze` label, this workflow will fail the PR check, blocking merges.
- 監控 PR 被加/移除 label、內容有更新（synchronize）時自動觸發。  
  It triggers automatically when labels are added/removed or the PR is synchronized.
- 判斷 PR 是否有 `merge-freeze` label，有的話就阻擋合併，並提示需要先移除這個 label 才能繼續。  
  If the PR has the `merge-freeze` label, merging is blocked and a message prompts to remove the label before merging.

---

# 2️⃣ Merge Freeze Labeler

**用途 / Purpose：**  
自動管理 `merge-freeze` label，讓「release」進行時，自動幫所有要進 main 的 PR 加上或移除 freeze 標記，減少人工作業，避免出錯。  
Automatically manage the `merge-freeze` label: when a release is in progress, all PRs targeting `main` are automatically labeled or unlabeled to reduce manual work and prevent mistakes.

**運作方式 / How it works：**

- 只要有 release 開頭（如 `release/v2025.06.06`）的 branch 開啟/同步/關閉 PR，這個 workflow 就會執行。  
  This workflow runs when a PR with a branch starting with `release/` is opened, synchronized, or closed.
- 當 release PR 開啟（或有內容更新）時，自動幫所有「目標是 main，來源不是 release 開頭」的 open PR 加上 `merge-freeze` 標籤，防止誤合併。  
  When a release PR is opened or updated, all open PRs targeting `main` (and not from a `release/` branch) are automatically labeled with `merge-freeze` to prevent accidental merges.
- 當 release PR 關閉（已 merge 或關閉）時，自動幫這些 PR 移除 `merge-freeze` 標籤，恢復合併權限。  
  When the release PR is closed (merged or closed), the label is removed from those PRs, allowing merges again.
- 這個流程只會在操作的 PR head branch 是 release 開頭時執行，不影響其他狀況。  
  This workflow only acts when the PR head branch starts with `release/` and does not affect other cases.
- 額外狀況： 當其他 non-release PR 有內容更新時，會去確認當前 PRs 有沒有 open 的 release PR，有的話就加上 `merge-freeze` 標籤，沒有的話就移除。  
  When other non-release PRs are updated, it checks if there are any open release PRs. If there are, it adds the `merge-freeze` label; if not, it removes the label.

---

## 用一句話總結 / In a nutshell

- **Labeler：** 自動加/移 `merge-freeze` 標籤，控制什麼時候 freeze。  
  Automatically add/remove the `merge-freeze` label to control when the freeze is active.
- **Blocker：** PR 有 freeze 標籤就擋住 merge，確保 main 只在適當時機能合併新東西。  
  Block merging if the PR has the freeze label, ensuring `main` only receives changes at the right time.
