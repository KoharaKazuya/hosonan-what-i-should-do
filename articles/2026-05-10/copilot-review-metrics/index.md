---
title: AI レビューを入れたら、次は「何を直せたか」を測りたい
slug: copilot-review-metrics
createdAt: 2026-05-10T05:25:34+00:00
updatedAt: 2026-05-10T05:25:34+00:00
---

AI コードレビューをチームに入れると、最初は「何件レビューされたか」「どれくらい使われたか」を見たくなります。導入初期の確認としては自然です。ただ、シニアエンジニアやエンジニアリングマネージャーが次に知りたいのは、レビューの量ではなく、どんな種類の問題が見つかり、そのうち何が実際に直されたかです。

GitHub は 2026 年 5 月 8 日、Copilot usage metrics API で Copilot code review の提案を comment type ごとに集計できるようにしました。`pull_requests` 配下に `copilot_suggestions_by_comment_type` が追加され、`security` や `bug_risk` のような分類ごとに、提案数と開発者が適用した提案数を見られると説明されています。対象は enterprise と organization の 1 日レポート、28 日ローリングレポートです。現時点ではリポジトリ単位のドリルダウンはできず、GitHub は将来に向けて調査中としています。[^1]

この更新で見るべき点は、「Copilot の管理画面に項目が増えた」ことではありません。AI レビューを、導入率の話からレビュー品質とチーム行動の話へ進める材料が増えていることです。

## 提案数だけでは、レビューの価値は分からない

レビューコメントが多いことは、必ずしも良いレビューを意味しません。細かい style 指摘が大量に出ているだけかもしれませんし、逆に少数でも `security` や `bug_risk` の指摘が高い確率で適用されているなら、チームには大きな価値があります。

GitHub の今回の説明では、comment type ごとに `total_copilot_suggestions` と `total_copilot_applied_suggestions` を返すとされています。つまり、AI が何を検出したかだけでなく、開発者がどの種類の指摘を受け入れたかを見られます。[^1] これは、AI レビューの評価を「コメントがあったか」から「レビューが実装変更につながったか」へ寄せる小さな変化です。

もちろん、適用率だけを KPI にするのも危険です。開発者が正しく無視した提案もありますし、手作業で別の修正をした場合は「適用」として見えない可能性があります。それでも、comment type 別の提案数と適用数を並べると、少なくとも次の問いを立てられます。

- `security` の提案は少ないが、適用率が高いのか
- `bug_risk` の提案は多いが、開発者に無視されがちなのか
- style や readability のような軽い指摘が、重要なレビューを埋もれさせていないか
- 特定のチームだけ、AI レビューを自動実行しているが実際には誰も触っていないのではないか

AI レビューを「人間レビューの代替」として見るより、「チームのレビューで何が検出され、何が修正に変換されているか」を可視化する補助線として見る方が、運用に落とし込みやすいです。

## 自動実行と、能動的な利用を分けて見る

GitHub は 2026 年 4 月にも、Copilot code review の利用者を active と passive に分けるメトリクスを追加しています。active は開発者が Copilot にレビューを依頼したり、再レビューを求めたり、提案を適用した場合です。passive はリポジトリや organization のポリシーによって自動レビューが走ったが、開発者が能動的に関与していない場合です。[^2]

さらに 2026 年 4 月 22 日には、enterprise と organization の 1 日レポート、28 日レポートで、active / passive の日次、週次、月次ユーザー数を集計できるようになりました。GitHub Docs でも、active usage は手動レビュー依頼または提案適用、passive usage は自動割り当てされたが能動的な関与がない状態として説明されています。[^3][^4]

この分離は、AI レビューの導入判断にかなり効きます。たとえば「全 PR に Copilot review を走らせているので導入率 100%」という状態でも、開発者が提案を読まず、適用もしていないなら、レビュー文化としては定着していません。逆に、自動実行の範囲は狭くても、開発者が `security` や `bug_risk` の提案を高い確率で適用しているなら、そこには改善余地ではなく成功パターンがあります。

見るべきなのは、AI レビューをオンにしたかではなく、開発者がどの場面で AI レビューを信頼し、どの場面で無視し、どの種類の指摘が人間レビューの前に修正されているかです。

## マネージャー向けの数字を、レビュー設計に戻す

Copilot usage metrics は 2026 年 2 月に GA となり、GitHub はダッシュボードと API で、利用傾向、コード生成、チャット、エージェント、PR 関連のメトリクスを見られると説明しています。API は enterprise、organization、user レベルのレポートを提供し、カスタムロールによる細かな閲覧権限やデータレジデンシーにも触れています。[^5]

ここで注意したいのは、メトリクスを「経営報告用の利用率」に閉じ込めないことです。GitHub Docs は、DAU、WAU、acceptance rate、agent adoption、モデル利用、言語利用などを見て、導入や enablement の改善に使う考え方を示しています。[^6] これは有用ですが、コードレビューではさらに実務寄りに戻す必要があります。

たとえば、comment type 別の適用率を見て、次のようにレビュー設計を変えられます。

- `security` の適用率が高いなら、該当領域では Copilot review を必須にし、人間レビューは設計判断や権限境界に集中する
- `bug_risk` の提案が多いのに適用率が低いなら、提案の質を人間がサンプリングし、誤検知なのか開発者が読んでいないのかを分ける
- 自動レビューの passive user が多いなら、全 PR 自動実行よりも、危険なディレクトリ、外部入力、認可、課金、DB migration などに対象を絞る
- 適用された提案が特定の言語やチームに偏るなら、プロンプト、レビュー基準、リポジトリの custom instructions を見直す

AI レビューのメトリクスは、チームを監視するためではなく、レビューの入口を設計し直すために使う方がよいです。どこで自動化し、どこで人間が読むべきかを、感覚ではなく実際の提案と適用の分布から決められます。

## 明日やるなら、この 3 つ

まず、Copilot code review を導入済みの organization で、active と passive の割合を確認します。自動レビューが走っているだけなのか、開発者が実際にレビュー依頼や提案適用をしているのかを分けます。GitHub Docs では、`used_agent` は IDE の agent mode を表し、Copilot code review は `used_copilot_code_review_active` / `used_copilot_code_review_passive` として別に扱われると説明されています。[^4]

次に、comment type ごとの提案数と適用数を 28 日単位で見ます。1 日ごとの揺れではなく、`security`、`bug_risk`、その他の分類で、どの指摘が実装変更に結びついているかを見ます。現時点ではリポジトリ単位に掘れないため、organization 全体の傾向として読み、気になる分類だけ人間が PR をサンプリングするのが現実的です。[^1]

最後に、AI レビューの目的を 1 つに絞って運用ルールを書きます。「全 PR を見せる」ではなく、「認可、外部入力、依存関係、DB migration を含む PR では Copilot review を必ず走らせ、`security` と `bug_risk` の提案は人間レビューで判断結果を残す」のように、対象と扱いを明確にします。

AI レビューは、コメント数を増やすための仕組みではありません。人間レビューの前に、機械が拾える危険信号をどれだけ修正に変換できるかを見る仕組みです。Copilot usage metrics の更新は、その評価を「使われたか」から「何を直せたか」に寄せる合図として扱うのがよさそうです。

現時点の留意点として、GitHub の 2026 年 5 月 8 日発表では comment type 別の集計は enterprise / organization レポート向けで、リポジトリ単位のドリルダウンはまだ提供されていません。また、comment type は Copilot が投稿時点で割り当てた分類であり、実際の修正価値を完全に表すものではありません。導入判断では、メトリクスと人間による PR サンプリングを組み合わせてください。[^1]

[^1]: GitHub Changelog, [Copilot code review comment types now in usage metrics API](https://github.blog/changelog/2026-05-08-copilot-code-review-comment-types-now-in-usage-metrics-api/)
[^2]: GitHub Changelog, [Copilot usage metrics now identify active and passive Copilot code review users](https://github.blog/changelog/2026-04-06-copilot-usage-metrics-now-identify-active-and-passive-copilot-code-review-users/)
[^3]: GitHub Changelog, [Copilot code review user counts now aggregate in usage metrics API](https://github.blog/changelog/2026-04-22-copilot-code-review-user-counts-now-aggregate-in-usage-metrics-api/)
[^4]: GitHub Docs, [Data available in Copilot usage metrics](https://docs.github.com/en/copilot/reference/copilot-usage-metrics/copilot-usage-metrics)
[^5]: GitHub Changelog, [Copilot metrics is now generally available](https://github.blog/changelog/2026-02-27-copilot-metrics-is-now-generally-available/)
[^6]: GitHub Docs, [Interpreting usage and adoption metrics for GitHub Copilot](https://docs.github.com/en/copilot/reference/copilot-usage-metrics/interpret-copilot-metrics)
