# AI--THON-2025
/*
Student Wellness Monitor
Single-file React component (App.jsx) built with Tailwind CSS and Recharts.

Features:
- Daily mood check-in (emoji + 1-10 scale + optional journal text)
- Simple sentiment analysis on journal text (lexicon-based)
- Personalized wellness recommendations based on mood & sentiment
- Trend visualization (line chart of mood scores over time)
- Tracking history stored in localStorage
- Export/Import JSON, simple CSV export
- Responsive layout and accessible form controls

How to use:
1. Create a React project (e.g. with Vite or Create React App).
2. Install dependencies:
   - npm install recharts
   - Tailwind CSS configured (or use your own CSS)
3. Place this file as src/App.jsx and run the app.

This is a single-file demo meant to be a working starting point you can extend.
*/

import React, { useEffect, useMemo, useState } from "react";
import {
  LineChart,
  Line,
  XAxis,
  YAxis,
  CartesianGrid,
  Tooltip,
  ResponsiveContainer,
  BarChart,
  Bar,
  Legend,
} from "recharts";

// --- Simple sentiment lexicon (tiny) ---
const POSITIVE = new Set([
  "good",
  "great",
  "happy",
  "joy",
  "excited",
  "calm",
  "relaxed",
  "thankful",
  "grateful",
  "optimistic",
]);
const NEGATIVE = new Set([
  "sad",
  "angry",
  "stress",
  "stressed",
  "depressed",
  "anxious",
  "worried",
  "tired",
  "lonely",
  "upset",
]);

function analyzeSentiment(text = "") {
  // Very small, explainable lexicon-based sentiment scoring.
  const words = text
    .toLowerCase()
    .replace(/[^a-z0-9\s]/g, " ")
    .split(" ")
    .filter(Boolean);
  if (words.length === 0) return { score: 0, label: "neutral" };
  let score = 0;
  for (const w of words) {
    if (POSITIVE.has(w)) score += 1;
    if (NEGATIVE.has(w)) score -= 1;
  }
  const normalized = Math.max(-1, Math.min(1, score / Math.max(1, words.length / 3)));
  const label = normalized > 0.1 ? "positive" : normalized < -0.1 ? "negative" : "neutral";
  return { score: normalized, label };
}

function moodToEmoji(score) {
  if (score >= 8) return "🤩";
  if (score >= 6) return "🙂";
  if (score >= 4) return "😐";
  if (score >= 2) return "😕";
  return "😢";
}

function getRecommendation(moodScore, sentimentLabel, journal) {
  // Basic rule-based recommendations. You can extend with ML models or user profile.
  const recs = [];
  if (moodScore >= 8 && sentimentLabel === "positive") {
    recs.push("You're doing great — keep the momentum! Try sharing gratitude with a friend.");
    recs.push("Maintain healthy habits: sleep, hydration, short walk.");
  } else if (moodScore >= 6) {
    recs.push("You're doing well — consider a mindfulness session (5-10 minutes).");
    recs.push("Reward yourself with a small break or fun activity.");
  } else if (moodScore >= 4) {
    recs.push("Try a breathing exercise (box breathing for 3 minutes).");
    recs.push("If possible, talk with a supportive friend or peer.");
  } else {
    recs.push("If you feel overwhelmed, try grounding exercises: name 5 things you can see.");
    recs.push("Consider reaching out to a counselor or trusted person.");
  }

  // If sentiment negative, add extra supportive suggestions
  if (sentimentLabel === "negative") {
    recs.push("Your journal shows negative sentiment; consider journaling for 5 minutes to process feelings.");
    recs.push("Try a gentle short walk or breathing exercise.");
  }

  // Keyword-based personalization
  if (/sleep|insomnia|tired/.test(journal.toLowerCase())) {
    recs.push("Sleep hygiene tip: maintain a consistent sleep schedule and reduce screen time 30 minutes before bed.");
  }
  if (/exam|test|project/.test(journal.toLowerCase())) {
    recs.push("Exam stress tip: break study into 25-minute focused sprints with 5-minute breaks (Pomodoro).");
  }

  // Return top 4 unique recommendations
  return Array.from(new Set(recs)).slice(0, 6);
}

function uid() {
  return Math.random().toString(36).slice(2, 9);
}

export default function App() {
  const [entries, setEntries] = useState(() => {
    try {
      const raw = localStorage.getItem("wellness_entries_v1");
      return raw ? JSON.parse(raw) : [];
    } catch (e) {
      return [];
    }
  });

  const [mood, setMood] = useState(7);
  const [journal, setJournal] = useState("");
  const [tags, setTags] = useState("");
  const [notePreview, setNotePreview] = useState(null);

  useEffect(() => {
    localStorage.setItem("wellness_entries_v1", JSON.stringify(entries));
  }, [entries]);

  const todayStr = useMemo(() => new Date().toISOString().slice(0, 10), []);

  function handleCheckIn(e) {
    e.preventDefault();
    const sentiment = analyzeSentiment(journal);
    const newEntry = {
      id: uid(),
      date: new Date().toISOString(),
      dateOnly: new Date().toISOString().slice(0, 10),
      mood,
      emoji: moodToEmoji(mood),
      journal,
      tags: tags
        .split(",")
        .map((t) => t.trim())
        .filter(Boolean),
      sentiment,
    };
    setEntries((s) => [newEntry, ...s].slice(0, 3650)); // keep up to ~10 years daily
    setJournal("");
    setTags("");
    setNotePreview(newEntry);
  }

  function clearAll() {
    if (!confirm("Clear all saved entries? This cannot be undone.")) return;
    setEntries([]);
    localStorage.removeItem("wellness_entries_v1");
  }

  function exportJSON() {
    const data = JSON.stringify(entries, null, 2);
    const blob = new Blob([data], { type: "application/json" });
    const url = URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.href = url;
    a.download = `wellness_entries_${new Date().toISOString().slice(0, 10)}.json`;
    a.click();
    URL.revokeObjectURL(url);
  }

  function exportCSV() {
    const header = ["date", "mood", "emoji", "journal", "tags", "sentiment_label", "sentiment_score"];
    const rows = entries.map((e) => [e.date, e.mood, e.emoji, (e.journal || "").replace(/\n/g, " "), (e.tags || []).join(";"), e.sentiment.label, e.sentiment.score]);
    const csv = [header, ...rows].map((r) => r.map((c) => `"${String(c).replace(/"/g, '""')}"`).join(",")).join("\n");
    const blob = new Blob([csv], { type: "text/csv" });
    const url = URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.href = url;
    a.download = `wellness_entries_${new Date().toISOString().slice(0, 10)}.csv`;
    a.click();
    URL.revokeObjectURL(url);
  }

  const chartData = useMemo(() => {
    // Prepare last 30 entries reversed (oldest → newest)
    const last = [...entries].slice(0, 90).reverse();
    return last.map((e) => ({ date: e.dateOnly, mood: e.mood }));
  }, [entries]);

  const aggregatedTags = useMemo(() => {
    const counts = {};
    entries.forEach((e) => {
      (e.tags || []).forEach((t) => {
        counts[t] = (counts[t] || 0) + 1;
      });
    });
    return Object.entries(counts).map(([tag, count]) => ({ tag, count }));
  }, [entries]);

  const averageMood = useMemo(() => {
    if (entries.length === 0) return "—";
    const sum = entries.reduce((a, b) => a + b.mood, 0);
    return (sum / entries.length).toFixed(2);
  }, [entries]);

  return (
    <div className="min-h-screen bg-gradient-to-b from-slate-50 to-white p-4 md:p-8">
      <div className="max-w-5xl mx-auto">
        <header className="flex items-center justify-between mb-6">
          <div>
            <h1 className="text-2xl md:text-3xl font-bold">Student Wellness Monitor</h1>
            <p className="text-sm text-slate-600">Daily check-ins, sentiment analysis, personalized recommendations, and trends.</p>
          </div>
          <div className="text-right">
            <div className="text-xs text-slate-500">Average mood</div>
            <div className="text-lg font-semibold">{averageMood} / 10</div>
          </div>
        </header>

        <main className="grid grid-cols-1 md:grid-cols-3 gap-6">
          {/* Left: Check-in form */}
          <section className="md:col-span-1 bg-white p-4 rounded-2xl shadow-sm">
            <form onSubmit={handleCheckIn}>
              <div className="mb-3">
                <label className="block text-sm font-medium mb-1">Today's mood</label>
                <div className="flex items-center gap-3">
                  <div className="text-2xl">{moodToEmoji(mood)}</div>
                  <input
                    type="range"
                    min={0}
                    max={10}
                    value={mood}
                    onChange={(e) => setMood(Number(e.target.value))}
                    className="w-full"
                    aria-label="mood slider"
                  />
                  <div className="w-10 text-right font-semibold">{mood}</div>
                </div>
              </div>

              <div className="mb-3">
                <label className="block text-sm font-medium mb-1">Quick journal (optional)</label>
                <textarea
                  value={journal}
                  onChange={(e) => setJournal(e.target.value)}
                  className="w-full rounded-md border-slate-200 shadow-sm p-2"
                  rows={4}
                  placeholder="How are you feeling? What happened today?"
                />
              </div>

              <div className="mb-3">
                <label className="block text-sm font-medium mb-1">Tags (comma separated)</label>
                <input
                  value={tags}
                  onChange={(e) => setTags(e.target.value)}
                  className="w-full rounded-md border-slate-200 shadow-sm p-2"
                  placeholder="e.g. exams, sleep, family"
                />
              </div>

              <div className="flex gap-2">
                <button
                  type="submit"
                  className="px-4 py-2 bg-indigo-600 text-white rounded-lg shadow hover:bg-indigo-700"
                >
                  Check in
                </button>

                <button
                  type="button"
                  onClick={() => {
                    const analysis = analyzeSentiment(journal);
                    alert(`Sentiment: ${analysis.label} (score: ${analysis.score.toFixed(2)})`);
                  }}
                  className="px-3 py-2 border rounded-lg"
                >
                  Analyze note
                </button>

                <button
                  type="button"
                  onClick={() => {
                    setJournal("");
                    setTags("");
                    setMood(7);
                  }}
                  className="px-3 py-2 border rounded-lg"
                >
                  Reset
                </button>
              </div>

              {notePreview && (
                <div className="mt-3 p-3 bg-slate-50 rounded">
                  <div className="text-xs text-slate-500">Last entry</div>
                  <div className="font-medium">{notePreview.emoji} {notePreview.mood}/10</div>
                  <div className="text-sm">{notePreview.journal || <em className="text-slate-400">(no journal)</em>}</div>
                  <div className="text-xs text-slate-500">sentiment: {notePreview.sentiment.label}</div>
                </div>
              )}
            </form>

            <div className="mt-4 border-t pt-3 text-sm text-slate-600">
              <button onClick={exportJSON} className="mr-2 underline">Export JSON</button>
              <button onClick={exportCSV} className="mr-2 underline">Export CSV</button>
              <button onClick={clearAll} className="text-red-600 underline">Clear all</button>
            </div>
          </section>

          {/* Middle: Recommendations & History */}
          <section className="md:col-span-2 bg-white p-4 rounded-2xl shadow-sm flex flex-col">
            <div className="mb-4 grid grid-cols-1 md:grid-cols-2 gap-4">
              <div>
                <h2 className="text-lg font-semibold">Personalized recommendations</h2>
                <div className="mt-2 text-sm text-slate-700">
                  {
                    // Create a live recommendation for the current draft journal
                  }
                  {getRecommendation(mood, analyzeSentiment(journal).label, journal).map((r, i) => (
                    <div key={i} className="mb-2 flex items-start gap-2">
                      <div className="w-6">•</div>
                      <div>{r}</div>
                    </div>
                  ))}
                </div>
              </div>

              <div>
                <h2 className="text-lg font-semibold">Recent history</h2>
                <div className="mt-2 text-sm text-slate-700 max-h-48 overflow-auto">
                  {entries.length === 0 && <div className="text-slate-400">No entries yet — check in today!</div>}
                  {entries.slice(0, 20).map((e) => (
                    <div key={e.id} className="flex items-start gap-3 py-2 border-b last:border-b-0">
                      <div className="text-2xl">{e.emoji}</div>
                      <div className="flex-1">
                        <div className="flex items-center justify-between">
                          <div className="font-medium">{e.mood}/10</div>
                          <div className="text-xs text-slate-500">{new Date(e.date).toLocaleString()}</div>
                        </div>
                        <div className="text-sm text-slate-700">{e.journal || <em className="text-slate-400">(no journal)</em>}</div>
                        <div className="text-xs text-slate-500">sentiment: {e.sentiment.label}</div>
                      </div>
                    </div>
                  ))}
                </div>
              </div>
            </div>

            <div className="mt-auto">
              <h3 className="text-sm font-medium">Tag cloud</h3>
              <div className="mt-2 flex flex-wrap gap-2">
                {aggregatedTags.length === 0 && <div className="text-slate-400">No tags yet</div>}
                {aggregatedTags.map((t) => (
                  <div key={t.tag} className="px-3 py-1 rounded-full bg-slate-100 text-sm">{t.tag} · {t.count}</div>
                ))}
              </div>
            </div>
          </section>

          {/* Bottom: Charts */}
          <section className="md:col-span-3 bg-white p-4 rounded-2xl shadow-sm">
            <div className="flex items-center justify-between mb-3">
              <h2 className="text-lg font-semibold">Mood trends</h2>
              <div className="text-sm text-slate-500">Showing up to last 90 entries</div>
            </div>

            <div style={{ height: 260 }}>
              <ResponsiveContainer width="100%" height="100%">
                <LineChart data={chartData} margin={{ top: 10, right: 20, left: 0, bottom: 0 }}>
                  <CartesianGrid strokeDasharray="3 3" />
                  <XAxis dataKey="date" minTickGap={10} />
                  <YAxis domain={[0, 10]} allowDecimals={false} />
                  <Tooltip />
                  <Line type="monotone" dataKey="mood" stroke="#4f46e5" strokeWidth={2} dot={{ r: 2 }} />
                </LineChart>
              </ResponsiveContainer>
            </div>

            <div className="mt-6 grid grid-cols-1 md:grid-cols-2 gap-4">
              <div className="p-3 bg-slate-50 rounded">
                <h4 className="text-sm font-medium mb-2">Mood distribution</h4>
                <div style={{ height: 180 }}>
                  <ResponsiveContainer width="100%" height="100%">
                    <BarChart data={aggregatedTags} layout="vertical" margin={{ top: 5, right: 20, left: 20, bottom: 5 }}>
                      <CartesianGrid strokeDasharray="3 3" />
                      <XAxis type="number" />
                      <YAxis dataKey="tag" type="category" />
                      <Tooltip />
                      <Bar dataKey="count" barSize={12} />
                    </BarChart>
                  </ResponsiveContainer>
                </div>
              </div>

              <div className="p-3 bg-slate-50 rounded">
                <h4 className="text-sm font-medium mb-2">Quick insights</h4>
                <ul className="text-sm list-disc list-inside text-slate-700">
                  <li>Entries saved: <strong>{entries.length}</strong></li>
                  <li>Average mood: <strong>{averageMood}</strong></li>
                  <li>Most recent: <strong>{entries[0] ? entries[0].dateOnly : "—"}</strong></li>
                </ul>
              </div>
            </div>
          </section>
        </main>

        <footer className="mt-6 text-center text-xs text-slate-500">Built with care • This is a demo implementation — not a substitute for professional help.</footer>
      </div>
    </div>
  );
}
