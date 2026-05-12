<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>11402 統計學(一)學期成績查詢 - 王建智 教授</title>

    <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>

    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@400;500;700&display=swap" rel="stylesheet">

    <style>
        body {
            font-family: 'Noto Sans TC', sans-serif;
        }

        @keyframes blink-pulse {
            0%, 100% {
                opacity: 1;
                transform: scale(1);
            }

            50% {
                opacity: 0.5;
                transform: scale(0.92);
            }
        }

        .animate-blink {
            animation: blink-pulse 1.2s ease-in-out infinite;
        }
    </style>
</head>

<body class="bg-slate-50">
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useEffect } = React;

        // Google Apps Script API 網址
        const API_URL = "https://script.google.com/macros/s/AKfycbyWgFR3wLz6hHPDJTtLSSzvwgFRrriiuzcCOwBsdc-HJvzP28PNashl3_S59fTAGZpqjQ/exec";

        const App = () => {
            const [query, setQuery] = useState('');
            const [searchResult, setSearchResult] = useState(null);
            const [loading, setLoading] = useState(false);
            const [stats, setStats] = useState({
                green: 0,
                yellow: 0,
                red: 0,
                total: 0
            });

            // ==============================
            // 燈號判斷規則
            // 四捨五入後：
            // 60 分以上：綠燈
            // 50～59 分：黃燈
            // 低於 50 分：紅燈
            // ==============================
            const getStatus = (score) => {
                const scoreValue = Math.round(parseFloat(score));

                if (scoreValue < 50) {
                    return {
                        color: 'red',
                        label: '紅燈',
                        desc: '成績未達標',
                        sub: '',
                        bg: 'bg-red-500'
                    };
                }

                if (scoreValue >= 60) {
                    return {
                        color: 'green',
                        label: '綠燈',
                        desc: '恭喜通過',
                        sub: '',
                        bg: 'bg-green-500'
                    };
                }

                return {
                    color: 'yellow',
                    label: '黃燈',
                    desc: '待加強',
                    sub: '(需申請補救)',
                    bg: 'bg-yellow-500'
                };
            };

            // ==============================
            // 畫面載入時取得班級統計
            // ==============================
            useEffect(() => {
                fetch(`${API_URL}?type=stats`)
                    .then(res => res.json())
                    .then(data => {
                        const total = data.green + data.yellow + data.red;
                        setStats({
                            green: data.green,
                            yellow: data.yellow,
                            red: data.red,
                            total: total
                        });
                    })
                    .catch(err => {
                        console.error("無法取得統計資料", err);
                    });
            }, []);

            // ==============================
            // 查詢個別學生
            // ==============================
            const handleSearch = () => {
                if (!query.trim()) return;

                setLoading(true);
                setSearchResult(null);

                fetch(`${API_URL}?q=${encodeURIComponent(query.trim())}`)
                    .then(res => res.json())
                    .then(data => {
                        console.log("API 回傳資料：", data);

                        if (data.status === 'success') {
                            setSearchResult(data.data);
                        } else {
                            setSearchResult('none');
                        }
                    })
                    .catch(err => {
                        alert("查詢發生錯誤，請稍後再試");
                        console.error(err);
                    })
                    .finally(() => {
                        setLoading(false);
                    });
            };

            return (
                <div className="min-h-screen p-4 md:p-8 flex flex-col items-center">
                    <div className="max-w-4xl w-full">

                        {/* 標題區 */}
                        <header className="text-center mb-10">
                            <h1 className="text-4xl font-bold text-slate-800 mb-3 tracking-tight">
                                11402 統計學(一)學期成績查詢
                            </h1>

                            <div className="inline-block bg-slate-800 text-white px-5 py-1.5 rounded-full text-sm font-medium shadow-md mb-2">
                                任課老師：王建智 教授
                            </div>

                            <p className="text-slate-500 text-sm mt-4">
                                請輸入學號（大寫 U 開頭）或姓名查詢個人狀態燈號
                            </p>
                        </header>

                        {/* 統計圖表 */}
                        {stats.total > 0 && (
                            <div className="bg-white p-6 rounded-3xl shadow-sm border border-slate-100 mb-10">
                                <h2 className="text-sm font-bold text-slate-400 uppercase tracking-widest mb-4 flex justify-between">
                                    <span>班級燈號分佈概況（共 {stats.total} 人）</span>
                                </h2>

                                <div className="flex h-5 w-full rounded-full overflow-hidden bg-slate-100 mb-6 border border-slate-50">
                                    <div
                                        style={{ width: `${(stats.green / stats.total) * 100}%` }}
                                        className="bg-green-500"
                                    ></div>

                                    <div
                                        style={{ width: `${(stats.yellow / stats.total) * 100}%` }}
                                        className="bg-yellow-500"
                                    ></div>

                                    <div
                                        style={{ width: `${(stats.red / stats.total) * 100}%` }}
                                        className="bg-red-500"
                                    ></div>
                                </div>

                                <div className="grid grid-cols-3 gap-4 text-center">
                                    <div className="p-2">
                                        <div className="text-2xl font-black text-green-500">
                                            {stats.green}
                                        </div>
                                        <div className="text-xs text-slate-500 font-bold">
                                            及格
                                        </div>
                                    </div>

                                    <div className="p-2 border-x border-slate-100">
                                        <div className="text-2xl font-black text-yellow-500">
                                            {stats.yellow}
                                        </div>
                                        <div className="text-xs text-slate-500 font-bold">
                                            待補救
                                        </div>
                                    </div>

                                    <div className="p-2">
                                        <div className="text-2xl font-black text-red-500">
                                            {stats.red}
                                        </div>
                                        <div className="text-xs text-slate-500 font-bold">
                                            不及格
                                        </div>
                                    </div>
                                </div>
                            </div>
                        )}

                        {/* 搜尋框 */}
                        <div className="flex flex-col items-center gap-6 mb-12">
                            <div className="flex w-full max-w-md shadow-2xl rounded-2xl overflow-hidden border-2 border-white focus-within:ring-4 focus-within:ring-blue-100 transition-all">
                                <input
                                    type="text"
                                    placeholder="輸入學號 或 姓名..."
                                    className="flex-grow px-6 py-4 outline-none text-lg"
                                    value={query}
                                    onChange={(e) => setQuery(e.target.value)}
                                    onKeyDown={(e) => e.key === 'Enter' && handleSearch()}
                                    disabled={loading}
                                />

                                <button
                                    onClick={handleSearch}
                                    disabled={loading}
                                    className={`bg-blue-600 text-white px-8 py-4 font-bold transition-colors ${
                                        loading
                                            ? 'opacity-50 cursor-not-allowed'
                                            : 'hover:bg-blue-700 active:scale-95'
                                    }`}
                                >
                                    {loading ? '查詢中...' : '查詢'}
                                </button>
                            </div>
                        </div>

                        {/* 結果顯示區 */}
                        <div className="min-h-[400px] flex items-center justify-center">

                            {searchResult === 'none' && !loading && (
                                <div className="text-center animate-bounce text-slate-400">
                                    <div className="text-5xl mb-4">🔍</div>
                                    <p className="text-lg font-bold">
                                        找不到資料，請確認輸入是否正確
                                    </p>
                                </div>
                            )}

                            {searchResult && searchResult !== 'none' && !loading && (() => {
                                const status = getStatus(searchResult.score);

                                return (
                                    <div className="bg-white w-full max-w-md rounded-[40px] shadow-2xl overflow-hidden border border-slate-100 flex flex-col md:flex-row animate-in fade-in zoom-in duration-300">

                                        {/* 左側燈號 */}
                                        <div className="bg-slate-900 p-8 flex flex-col gap-6 items-center justify-center">

                                            <div className={`relative w-16 h-16 rounded-full border-4 border-slate-800 flex items-center justify-center transition-all duration-700 ${
                                                status.color === 'red'
                                                    ? 'bg-red-500 shadow-[0_0_35px_rgba(239,68,68,1)]'
                                                    : 'bg-slate-800 opacity-10'
                                            }`}>
                                                {status.color === 'red' && (
                                                    <div className="w-16 h-16 rounded-full bg-red-500 animate-blink absolute"></div>
                                                )}
                                            </div>

                                            <div className={`relative w-16 h-16 rounded-full border-4 border-slate-800 flex items-center justify-center transition-all duration-700 ${
                                                status.color === 'yellow'
                                                    ? 'bg-yellow-500 shadow-[0_0_35px_rgba(234,179,8,1)]'
                                                    : 'bg-slate-800 opacity-10'
                                            }`}>
                                                {status.color === 'yellow' && (
                                                    <div className="w-16 h-16 rounded-full bg-yellow-500 animate-blink absolute"></div>
                                                )}
                                            </div>

                                            <div className={`relative w-16 h-16 rounded-full border-4 border-slate-800 flex items-center justify-center transition-all duration-700 ${
                                                status.color === 'green'
                                                    ? 'bg-green-500 shadow-[0_0_35px_rgba(34,197,94,1)]'
                                                    : 'bg-slate-800 opacity-10'
                                            }`}>
                                                {status.color === 'green' && (
                                                    <div className="w-16 h-16 rounded-full bg-green-500 animate-blink absolute"></div>
                                                )}
                                            </div>
                                        </div>

                                        {/* 右側學生資訊 */}
                                        <div className="p-10 flex-grow flex flex-col justify-center bg-white text-center md:text-left">
                                            <span className="text-blue-500 text-xs font-black uppercase tracking-widest mb-2">
                                                Individual Report
                                            </span>

                                            <h3 className="text-4xl font-black text-slate-800 mb-1">
                                                {searchResult.name}
                                            </h3>

                                            <p className="text-slate-400 font-mono text-sm mb-2 tracking-tighter italic">
                                                {searchResult.id}
                                            </p>

                                            {/* 顯示系統實際讀取分數 */}
                                            <p className="text-slate-500 text-sm font-bold mb-8">
                                                系統讀取成績：{searchResult.score}

                                                {searchResult.rawScore !== undefined && (
                                                    <span className="block text-xs text-slate-400 mt-1">
                                                        原始成績：{searchResult.rawScore}
                                                    </span>
                                                )}
                                            </p>

                                            <div className="border-t border-slate-100 pt-8">
                                                <p className="text-xs text-slate-400 font-bold mb-2 uppercase">
                                                    Study Progress
                                                </p>

                                                <div className={`mb-6 ${
                                                    status.color === 'red'
                                                        ? 'text-red-500'
                                                        : status.color === 'green'
                                                            ? 'text-green-500'
                                                            : 'text-yellow-600'
                                                }`}>

                                                    <div className="text-4xl font-black leading-tight">
                                                        {status.desc}
                                                    </div>

                                                    {status.sub && (
                                                        <div className="text-xl font-bold mt-1 opacity-80">
                                                            {status.sub}
                                                        </div>
                                                    )}
                                                </div>

                                                <div className="mt-4 p-4 bg-slate-50 rounded-2xl border border-slate-100">
                                                    <p className="text-xs text-slate-500 leading-relaxed font-medium">
                                                        {status.color === 'yellow'
                                                            ? "您的學習狀態目前為黃燈。請確認是否需要參加補救教學，並主動聯繫教授討論。"
                                                            : status.color === 'red'
                                                                ? "您的學習狀態目前為紅燈。本學期成績未達標準，請留意後續相關重修規定。"
                                                                : "恭喜！您的學習狀態為綠燈。本學期成績已達通過標準，請繼續保持！"
                                                        }
                                                    </p>
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                );
                            })()}
                        </div>
                    </div>
                </div>
            );
        };

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>
