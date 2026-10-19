<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>小柳ロウ 配信スケジュール（自動取得）</title>
    <style>
        /* 小柳ロウさんをイメージしたダーク＆クールなデザイン */
        body { font-family: 'Helvetica Neue', Arial, sans-serif; background-color: #1a1c23; color: #e2e8f0; text-align: center; padding: 20px; margin: 0; }
        h1 { color: #818cf8; margin-bottom: 5px; font-size: 1.8rem; }
        .subtitle { color: #94a3b8; font-size: 0.9rem; margin-bottom: 30px; }
        .loading { font-size: 1.2rem; color: #a5b4fc; margin-top: 50px; }
        .schedule-container { max-width: 600px; margin: 0 auto; display: flex; flex-direction: column; gap: 15px; }
        .schedule-card { background: #2d3142; border-left: 5px solid #6366f1; padding: 15px; border-radius: 6px; text-align: left; box-shadow: 0 4px 6px rgba(0,0,0,0.2); display: flex; justify-content: space-between; align-items: center; }
        .schedule-card.live { border-left-color: #ef4444; background: #352635; }
        .status-badge { background: #475569; color: #fff; font-size: 0.75rem; padding: 2px 6px; border-radius: 4px; font-weight: bold; margin-bottom: 5px; display: inline-block; }
        .status-badge.live { background: #ef4444; }
        .info-zone { width: 70%; }
        .time-text { font-size: 1.1rem; font-weight: bold; color: #f8fafc; }
        .title-text { font-size: 0.95rem; color: #cbd5e1; margin-top: 5px; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
        .btn-zone { width: 25%; text-align: right; }
        .link-btn { display: inline-block; padding: 8px 12px; background: #4f46e5; color: white; text-decoration: none; border-radius: 4px; font-size: 0.85rem; font-weight: bold; transition: 0.2s; }
        .link-btn:hover { background: #4338ca; }
        .no-data { color: #94a3b8; margin-top: 5px; }
    </style>
</head>
<body>

    <h1>小柳ロウ 配信スケジュール</h1>
    <div class="subtitle">にじさんじ公式サイトより自動取得中</div>

    <div id="loading" class="loading">スケジュールを読み込み中...</div>
    <div id="schedule-list" class="schedule-container" style="display: none;"></div>

    <script>
        // 小柳ロウさんのにじさんじ公式ライバーID
        const LIVER_ID = "rou-koyanagi";
        
        // にじさんじ公式サイトが使用しているスケジュールAPIエンドポイント
        const API_URL = `https://nijisanji.jp{LIVER_ID}/events`;

        async function fetchSchedule() {
            try {
                const response = await fetch(API_URL);
                if (!response.ok) throw new Error("データの取得に失敗しました");
                
                const data = await response.json();
                // 配信イベントデータのみを抽出 (直近のもの)
                const events = data.events || [];
                
                // 現在時刻より前の終了した配信を除外したい場合はここでフィルター可能
                // 今回は公式から返ってくる直近スケジュールをそのまま表示します
                
                displaySchedule(events);
            } catch (error) {
                console.error(error);
                document.getElementById('loading').innerText = "スケジュールの取得エラーが発生しました。";
            }
        }

        function displaySchedule(events) {
            const loadingEl = document.getElementById('loading');
            const listEl = document.getElementById('schedule-list');
            
            loadingEl.style.display = 'none';
            listEl.style.display = 'flex';
            listEl.innerHTML = '';

            if (events.length === 0) {
                listEl.innerHTML = '<p class="no-data">現在、登録されている直近の配信予定はありません。</p>';
                return;
            }

            events.forEach(event => {
                // 日時のフォーマット整形
                const startTime = new Date(event.start_at);
                const month = startTime.getMonth() + 1;
                const date = startTime.getDate();
                const hours = String(startTime.getHours()).padStart(2, '0');
                const minutes = String(startTime.getMinutes()).padStart(2, '0');
                const timeString = `${month}/${date} ${hours}:${minutes} 〜`;

                // 配信ステータス (配信中か、予定か)
                const isLive = event.is_live; 
                const badgeText = isLive ? "LIVE" : "配信予定";
                const badgeClass = isLive ? "status-badge live" : "status-badge";
                const cardClass = isLive ? "schedule-card live" : "schedule-card";

                // カードのHTML生成
                const card = document.createElement('div');
                card.className = cardClass;
                card.innerHTML = `
                    <div class="info-zone">
                        <span class="${badgeClass}">${badgeText}</span>
                        <div class="time-text">${timeString}</div>
                        <div class="title-text" title="${event.title || 'タイトル未設定'}">${event.title || '配信枠（未定）'}</div>
                    </div>
                    <div class="btn-zone">
                        <a href="${event.url}" class="link-btn" target="_blank">YouTube</a>
                    </div>
                `;
                listEl.appendChild(card);
            });
        }

        // ページ読み込み時に実行
        fetchSchedule();
    </script>
</body>
</html>
