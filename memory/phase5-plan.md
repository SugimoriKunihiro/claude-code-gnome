# Phase 5: Frontend + API — 完了 + 微修正残

## 完了した作業 (2026-04-19)
- [x] Vite + React 18 フロントエンド
- [x] エージェント切替 (🦊🌙🌻🍀) + scaling (nano/mini/full)
- [x] ChatWindow + ReactMarkdown + CodeBlock(Copy) + remark-gfm(URLリンク化)
- [x] PTTButton (Push-to-Talk, Whisper STT)
- [x] VRMWidget (旧実装そのまま、フローティング小窓)
- [x] api_server.py (FastAPI, Lightsail同居, port 8091)
- [x] Frontend dist → Lightsail配信
- [x] API Gateway HTTPS proxy → Portal iframe
- [x] 会話履歴DynamoDBロード (全4エージェント)
- [x] エージェント別非同期送受信
- [x] フッター分離 (result + footer 別フィールド)
- [x] コピーボタン (メッセージ下部)
- [x] ダークモード対応 (リンク色, UI全般)
- [x] Lambda GETリクエスト空Buffer修正

## 残りの微修正
- [ ] WF完了遷移の検証 (WF016→OK→addPrayerRecord が時々動かない)
- [ ] ルーチン9/9完了時の自動終了検証 (tool observation追加済み、要テスト)
- [ ] P1: 多重送信防止の強化 (debounce)
- [ ] Lightsail api_server systemd化 (再起動時に自動起動)
- [ ] Frontend rebuild自動化スクリプト
- [ ] Portal submodule commit
- [ ] VRMアバターのemotion/lipsync実動作テスト
- [ ] スマホ対応テスト (Portal iframe内)

## 本番環境
- Portal: https://sugimorikunihiro.github.io/SSA/ (GitHub Pages)
- Frontend: https://lxo7uvj8r0.execute-api.ap-northeast-1.amazonaws.com/ (API Gateway → Lightsail)
- API: http://13.113.40.114:8091/api/* (Lightsail直)
