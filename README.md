# AI--THON-2025
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
