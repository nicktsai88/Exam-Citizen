# Firebase 設定｜Exam-Citizen-a-7

本測驗固定使用集合 `Exam-Citizen-a-7`、文件 `global` 與 Firebase 匿名驗證；網頁不會初始化 Analytics。

## 啟用步驟

1. 在 Firebase Console 開啟專案 `english-reading-6b506`。
2. 到 **Authentication → Sign-in method** 啟用 **Anonymous／匿名**。
3. 建立 Cloud Firestore，選擇合適區域。
4. 到 **Authentication → Settings → Authorized domains** 加入實際 HTTPS 部署網域（例如 GitHub Pages 網域）。
5. Firebase 功能請用 HTTPS 網址或 localhost 測試。`file://` 只會使用訪客／本機模式。

匿名驗證只提供 Firestore 工作階段；暱稱學習者不是密碼保護的私人帳號。

## Firestore Security Rules

把下列 helper functions 與 `match /Exam-Citizen-a-7/global` 合併到既有 `match /databases/{database}/documents` 中。請保留其他測驗的規則，不要整份覆蓋；正式使用前先在 Rules Playground 測試。

```text
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    function isAnonymous() {
      return request.auth != null
        && request.auth.token.firebase.sign_in_provider == 'anonymous';
    }

    function validAvatar(value) {
      return value in ['cat', 'dog', 'rabbit', 'bear', 'fox', 'panda'];
    }

    function validLearner(learnerId, data) {
      return data.keys().hasOnly([
          'learnerId', 'nickname', 'avatarId',
          'createdAt', 'updatedAt', 'lastActiveAt'
        ])
        && data.keys().hasAll([
          'learnerId', 'nickname', 'avatarId',
          'createdAt', 'updatedAt', 'lastActiveAt'
        ])
        && data.learnerId == learnerId
        && data.nickname is string
        && data.nickname.size() >= 1
        && data.nickname.size() <= 20
        && data.avatarId is string
        && validAvatar(data.avatarId)
        && data.createdAt is timestamp
        && data.updatedAt is timestamp
        && data.lastActiveAt is timestamp;
    }

    function validProgress(learnerId, data) {
      return data.keys().hasOnly([
          'learnerId', 'articleTitle', 'quizVersion', 'attemptId',
          'practiceType', 'activeQuestionIds', 'answers', 'locked',
          'answeredAt', 'wrongQuestionIds', 'submitted',
          'retainedCorrectFromAttemptId', 'retainedCorrectCount',
          'startedAt', 'completedAt', 'resetAt',
          'clientMutationId', 'operation', 'revision', 'updatedAt'
        ])
        && data.keys().hasAll([
          'learnerId', 'articleTitle', 'quizVersion', 'attemptId',
          'practiceType', 'activeQuestionIds', 'answers', 'locked',
          'answeredAt', 'wrongQuestionIds', 'submitted',
          'retainedCorrectFromAttemptId', 'retainedCorrectCount',
          'startedAt', 'completedAt', 'resetAt',
          'clientMutationId', 'operation', 'revision', 'updatedAt'
        ])
        && data.learnerId == learnerId
        && data.articleTitle == '大滿貫公民｜地方政府與政治參與'
        && data.quizVersion == 'a7-ui-2026.11'
        && data.attemptId is string
        && data.attemptId.size() >= 1
        && data.attemptId.size() <= 128
        && data.practiceType in ['full', 'wrong-review']
        && data.activeQuestionIds is list
        && data.answers is map
        && data.locked is map
        && data.answeredAt is map
        && data.wrongQuestionIds is list
        && data.submitted is bool
        && (data.retainedCorrectFromAttemptId == null
          || data.retainedCorrectFromAttemptId is string)
        && data.retainedCorrectCount is int
        && data.retainedCorrectCount >= 0
        && (data.startedAt == null || data.startedAt is timestamp)
        && (data.completedAt == null || data.completedAt is timestamp)
        && (data.resetAt == null || data.resetAt is timestamp)
        && data.clientMutationId is string
        && data.clientMutationId.size() >= 1
        && data.operation is string
        && data.revision is int
        && data.revision >= 1
        && data.updatedAt is timestamp;
    }

    function validAttempt(learnerId, attemptId, data) {
      return data.keys().hasOnly([
          'attemptId', 'articleTitle', 'quizVersion', 'learnerId',
          'nicknameSnapshot', 'avatarIdSnapshot', 'practiceType',
          'retainedCorrectFromAttemptId', 'retainedCorrectCount',
          'startedAt', 'completedAt', 'totalQuestions',
          'attemptedQuestions', 'correctCount', 'wrongCount', 'score',
          'questionIds', 'wrongQuestionIds', 'clientMutationId', 'savedAt'
        ])
        && data.keys().hasAll([
          'attemptId', 'articleTitle', 'quizVersion', 'learnerId',
          'nicknameSnapshot', 'avatarIdSnapshot', 'practiceType',
          'retainedCorrectFromAttemptId', 'retainedCorrectCount',
          'startedAt', 'completedAt', 'totalQuestions',
          'attemptedQuestions', 'correctCount', 'wrongCount', 'score',
          'questionIds', 'wrongQuestionIds', 'clientMutationId', 'savedAt'
        ])
        && data.learnerId == learnerId
        && data.attemptId == attemptId
        && data.articleTitle == '大滿貫公民｜地方政府與政治參與'
        && data.quizVersion == 'a7-ui-2026.11'
        && data.nicknameSnapshot is string
        && data.nicknameSnapshot.size() >= 1
        && data.nicknameSnapshot.size() <= 20
        && (data.avatarIdSnapshot == 'guest'
          || validAvatar(data.avatarIdSnapshot))
        && data.practiceType in ['full', 'wrong-review']
        && (data.retainedCorrectFromAttemptId == null
          || data.retainedCorrectFromAttemptId is string)
        && data.retainedCorrectCount is int
        && data.retainedCorrectCount >= 0
        && (data.startedAt == null || data.startedAt is timestamp)
        && data.completedAt is timestamp
        && data.totalQuestions is int
        && data.totalQuestions > 0
        && data.attemptedQuestions is int
        && data.attemptedQuestions == data.totalQuestions
        && data.correctCount is int
        && data.wrongCount is int
        && data.correctCount >= 0
        && data.wrongCount >= 0
        && data.correctCount + data.wrongCount == data.totalQuestions
        && (data.score is int || data.score is float)
        && data.score >= 0
        && data.score <= 100
        && data.questionIds is list
        && data.wrongQuestionIds is list
        && data.clientMutationId is string
        && data.clientMutationId.size() >= 1
        && data.savedAt is timestamp;
    }

    match /Exam-Citizen-a-7/global {
      allow read: if isAnonymous();
      allow write: if false;

      match /learners/{learnerId} {
        allow read: if isAnonymous();
        allow create: if isAnonymous()
          && validLearner(learnerId, request.resource.data);
        allow update: if isAnonymous()
          && validLearner(learnerId, request.resource.data)
          && request.resource.data.createdAt == resource.data.createdAt;
        allow delete: if false;

        match /attempts/{attemptId} {
          allow read: if isAnonymous();
          allow create: if isAnonymous()
            && validAttempt(learnerId, attemptId, request.resource.data);
          allow update: if isAnonymous()
            && validAttempt(learnerId, attemptId, request.resource.data)
            && request.resource.data.clientMutationId
              == resource.data.clientMutationId
            && request.resource.data.diff(resource.data).affectedKeys()
              .hasOnly(['savedAt']);
          allow delete: if false;
        }
      }

      match /progress/{learnerId} {
        allow read: if isAnonymous();
        allow create: if isAnonymous()
          && validProgress(learnerId, request.resource.data)
          && request.resource.data.revision == 1;
        allow update: if isAnonymous()
          && validProgress(learnerId, request.resource.data)
          && (
            request.resource.data.revision == resource.data.revision + 1
            || (
              request.resource.data.revision == resource.data.revision
              && request.resource.data.clientMutationId
                == resource.data.clientMutationId
            )
          );
        allow delete: if false;
      }
    }
  }
}
```

## 正式驗收

- 用兩個獨立瀏覽器工作階段開啟同一 HTTPS 網址，選取同一張學習者卡並確認簡短識別碼相同。
- 一端作答後，另一端應無須重新整理即可更新答案、鎖定、正誤、統計與題號狀態。
- 只切換解析／測驗、調整字級或捲動時，Firestore 不應出現新的作答寫入。
- 測試斷線作答、恢復連線、背景返回、同時作答、空白新裝置、重新開始、錯題重練與重複交卷。
- 只有 Firestore 寫入完成後，網頁才應顯示「✅ 已同步雲端」。

共享暱稱資料對所有能開啟頁面並通過匿名驗證的人可見。若未來需要私人帳號，必須改用真正登入，並以 `request.auth.uid` 隔離資料。
