---
title: "Cloudflare WARP（1.1.1.1）× macOS：DNS・SOCKS・Local Proxyを腹落ちさせるベストプラクティス"
emoji: "🐸"
type: "tech"
topics: ["cloudflare", "warp", "macos", "dns", "proxy", "network"]
published: false
---

## はじめに：WARPを使うと「DNS設定って要るの？」が気になり出す

Cloudflare One（WARP / 1.1.1.1）をONにしていると、macOS側の「DNS」や「プロキシ」設定がどこまで効いているのかが急に曖昧になります。

- DNSサーバー欄は `1.1.1.1` 以外消していいの？
- そもそも WARP をONにしてるならDNS設定は不要？
- WARPの「WARP via Local Proxy」って何？
- macの SOCKS プロキシって何者？
- 「接続先がCloudflare上だとVPNとして意味がない」って本当？

この記事は、上の疑問を **一つのストーリーとして整理して、最短で“腹落ち”させる**ためのまとめです。

## 1. まず結論：WARPがONなら「macOSのDNS設定」は基本的に主役じゃない

### WARP（VPN/トンネル）がONの間に起きていること

ざっくりこうです。

```
アプリ → WARP（仮想インターフェース）→ Cloudflare → インターネット
```

この時点でDNSも含めて「入口」がWARP側に切り替わるため、macOSのネットワーク設定画面にあるDNS（Wi-FiのDNS）をいじっても、挙動が変わらないことが多いです。

> 重要：DNS設定が「不要」になるというより、WARP ON 中は「参照されにくい（影響が小さい）」が正確。

## 2. DNSサーバー欄の「1.1.1.1 以外」は消していい？

ここは「何を優先したいか」で決まります。

### 2-1. WARPを常時ON運用するなら：消しても問題が出にくい

WARPがONなら、macOS側のDNS設定が使われにくいので、Wi-FiのDNSに `1.1.1.1` を残さなくても大勢に影響はありません。

- 目的：WARPに一任する
- 方針：Wi-Fi側DNSは「保険」扱い

### 2-2. ただし「WARPがOFFになる瞬間」だけはWi-Fi側DNSが効く

例えば：

- WARPをOFFにした
- WARPが落ちた / 接続失敗した
- 一時停止した

この瞬間、OSは通常のネットワーク経路に戻ります。ここでWi-Fi側DNSの設定が“復活”します。

## 3. macOSでSOCKSって何？（＝万能な“中継係”）

SOCKS（ソックス）は一言で言うと：

> 通信の中身を解釈せずに、全部まとめて別の出口へ流す中継（プロキシ）

### 3-1. HTTPプロキシとの違い

| 種類 | 何に強い？ | 中身を見る？ | 汎用性 |
|---|---|---|---|
| HTTP/HTTPSプロキシ | Web中心 | ある程度見る | 限定的 |
| SOCKSプロキシ | ほぼ何でも | 基本見ない | 高い |

SOCKSは「これはHTTPです」「これはDNSです」みたいな解釈をしないので、仕組みとしてシンプルで強い。

## 4. WARPの3モード：「1.1.1.1」「1.1.1.1 with WARP」「WARP via Local Proxy」の違い

WARPアプリの表示に出てくるモードは“レイヤー”が違います。

### 4-1. 1.1.1.1（DNS only）

- DNSだけCloudflareへ
- 通信全体は通常回線のまま

```
アプリ → 通常回線 → インターネット
        └ DNSだけ → 1.1.1.1
```

### 4-2. 1.1.1.1 with WARP（WARP VPN）

- DNS＋通信全体をトンネル
- いわゆるVPNに近い使い方

```
アプリ → WARP → Cloudflare → インターネット
```

### 4-3. WARP via Local Proxy（ローカルSOCKSプロキシ経由）

- WARPがローカルにSOCKS5（例：127.0.0.1:40000）を立てる
- macOS側がそこに流す

```
アプリ → SOCKS(127.0.0.1:40000) → WARP → Cloudflare → インターネット
```

「どこを通ってるか」が明確で、検証・制御がしやすい。上級者が好む構成。

## 5. スクショ構成の評価：WARP Proxy Mode + macOS SOCKS = 整合している

あなたが設定した構成は、こういう対応関係でした。

- WARP：Proxy mode ON（Port 40000）
- macOS：SOCKSプロキシ ON（127.0.0.1:40000）
- HTTP/HTTPSプロキシ：OFF

これは矛盾がなく、意図が明確です。

### 5-1. 注意点（必ず知っておく）

WARPをOFFにしたまま、macOSのSOCKSがONだと：

- 127.0.0.1:40000 が“出口不在”になる
- 通信できない / 一部アプリが止まる

**運用ルール：**
- WARPをOFFにするなら、macOSのSOCKSもOFFにする
（常時ON運用ならこの問題は起きない）

## 6. 「Cloudflare上のサービスに接続するなら、WARPはVPNとして機能しない」は本当？

### 6-1. 正確には「機能しない」ではなく「VPNっぽい効果が見えにくい」

WARPは「Cloudflareネットワークに最短で乗せる」思想のため、相手がCloudflare上だとこうなりがちです。

```
あなた → WARP → Cloudflare Edge →（目的地もCloudflare）→ 完了
```

入口も出口もCloudflareだと「外に出た感」が薄くなります。

- IPが変わった実感がない
- 経路が大きく変わらない
- そもそも相手側がCloudflareで保護されている

この体感が「WARP意味なくない？」という誤解につながります。

### 6-2. それでも無意味ではない

Cloudflare宛てでも、少なくとも次は成立します。

- ローカルネットワーク（公衆Wi-Fi等）からの覗き見対策
- DNSや通信の保護（経路上の盗聴耐性）

## 7. ベストプラクティス：macOS×WARP運用のおすすめパターン

### パターンA：迷ったらこれ（一般最適解）
- モード：**1.1.1.1 with WARP**
- macOS：DNS/プロキシは基本いじらない

> “VPNっぽく”素直に使う一番ラクな構成。

### パターンB：DNSだけ整えたい（VPN不要）
- モード：**1.1.1.1（DNS only）**
- macOS：DNSをCloudflareに（必要なら）

> 最小コストでDNSのプライバシーだけ取る。

### パターンC：経路を完全に把握したい（上級・検証向け）
- モード：**WARP via Local Proxy**
- macOS：**SOCKS（127.0.0.1:40000）ON**
- HTTP/HTTPS：OFF

> 「どこを通っているか」が明確で、DNS設定をあれこれ触らなくて済む。

## 8. まとめ：設定の“主役”を取り違えない

- WARPがONなら、主役は「WARP経路」
- macOSのDNSは「WARP OFF 時の保険」
- SOCKSは「全部まとめて流す万能中継」
- Cloudflare宛てでVPN効果が薄く見えても「止まってる」わけではない
