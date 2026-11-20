# projet-GABY-import { teacherHas } from "@/lib/gate-permissions";
import { createFileRoute, Link } from "@tanstack/react-router";
import { useMemo, useState } from "react";
import { AccessGate } from "@/components/AccessGate";
import { AcademicBackground } from "@/components/AcademicBackground";
import { Button } from "@/components/ui/button";
import { FounderInbox } from "@/components/FounderInbox";
import { PhyloCountdown } from "@/components/PhyloCountdown";
import { SuggestionBox } from "@/components/SuggestionBox";
import { ChatBox } from "@/components/ChatBox";
import { PrivateChat } from "@/components/PrivateChat";
import { TeachersPanel } from "@/components/TeachersPanel";
import { FounderAlerts } from "@/components/FounderAlerts";
import { useAccess } from "@/hooks/useAccess";
import { subjects } from "@/lib/courses";
import { getSubjectTheme } from "@/lib/subjectTheme";


export const Route = createFileRoute("/")({
  head: () => ({
    meta: [
      { title: "Gaby Rapide — tous tes cours de Terminale" },
      {
        name: "description",
        content:
          "Espace de révision protégé par code : Enseignement scientifique, Phylo, Histoire, SES, Hébreu, Anglais et HGGSP en un clic.",
      },
      { property: "og:title", content: "Gaby Rapide — tous tes cours de Terminale" },
      {
        property: "og:description",
        content: "Sept compartiments de cours, un seul code d'accès. Révise vite et bien.",
      },
      { property: "og:type", content: "website" },
      { name: "twitter:card", content: "summary_large_image" },
    ],
  }),
  component: Index,
});

const tileStyles = [
  "", "", "", "", "", "", "col-span-2 lg:col-span-3",
];

function Index() {
  const { unlocked, ready, refresh, lock, founder, teacher, audience } = useAccess();
  const [query, setQuery] = useState("");

  const visibleSubjects = teacher ? subjects.filter((subject) => teacherHas(teacher, subject.slug)) : subjects;

  const results = useMemo(() => {
    const q = query.trim().toLowerCase();
    if (!q) return [];
    return visibleSubjects.flatMap((s) =>
      s.courses
        .filter(
          (c) =>
            c.title.toLowerCase().includes(q) ||
            c.summary.toLowerCase().includes(q) ||
            s.name.toLowerCase().includes(q),
        )
        .map((c) => ({ subject: s, course: c })),
    );
  }, [query, teacher]);

  if (!ready) return <div className="min-h-screen bg-paper" />;
  if (!unlocked) return <AccessGate onUnlock={refresh} />;

  return (
    <main className="academic-interior relative min-h-screen bg-paper text-ink">
      <AcademicBackground />
      <section className="relative z-10 max-w-6xl mx-auto px-6 py-12">
        <div className="flex flex-wrap gap-4 items-end justify-between border-b border-border pb-6">
          <div>
            <div className="font-mono text-[11px] uppercase tracking-[0.25em] text-mute">
              Compartiments — {subjects.length}
            </div>
            <h1 className="academic-title mt-2 text-3xl font-bold leading-none">
              Gaby Rapide
            </h1>
          </div>
          {audience !== "autre" && <Link to="/devoirs" className="rounded border border-primary px-4 py-2 text-sm font-bold uppercase tracking-widest text-primary hover:bg-primary hover:text-primary-foreground">
            Devoirs
          </Link>}
          <div className="flex items-center gap-2 text-sm rounded border border-input bg-background px-3 py-2">
            <span className="text-acc">/</span>
            <input
              value={query}
              onChange={(e) => setQuery(e.target.value)}
              placeholder="Rechercher un cours…"
              className="bg-transparent text-foreground placeholder:text-muted-foreground outline-none w-48 sm:w-56"
            />
          </div>
        </div>

        {query.trim() ? (
          <ul className="mt-8">
            {results.length === 0 && (
              <li className="font-mono text-sm text-mute py-6">Aucun cours trouvé.</li>
            )}
            {results.map(({ subject, course }) => (
              <li key={course.id}>
                <Link
                  to="/matiere/$slug"
                  params={{ slug: subject.slug }}
                  className="group flex items-center gap-4 py-4 border-b border-line hover:border-ink transition-colors"
                >
                  <span className="font-mono text-xs text-mute w-32 shrink-0 uppercase tracking-widest truncate">
                    {subject.short}
                  </span>
                  <span className="flex-1 text-lg font-medium truncate">{course.title}</span>
                  <span className="font-mono text-xs uppercase tracking-widest text-mute group-hover:text-acc">
                    Ouvrir →
                  </span>
                </Link>
              </li>
            ))}
          </ul>
        ) : (
          <div className="grid grid-cols-2 lg:grid-cols-3 gap-4 mt-8">
            {visibleSubjects.map((subject, i) => (
              <Link
                key={subject.slug}
                to="/matiere/$slug"
                params={{ slug: subject.slug }}
                className={`academic-subject group relative overflow-hidden rounded-lg border border-border bg-panel text-foreground p-5 min-h-[190px] flex flex-col justify-between rise ${tileStyles[i]}`}
                style={{ animationDelay: `${60 + i * 60}ms` }}
              >
                 <img src={getSubjectTheme(subject.slug).image} alt="" loading="lazy" width={1536} height={768} className="absolute inset-0 h-full w-full object-cover" />
                 <div className="subject-tile-shade absolute inset-0" aria-hidden />
                 <div className="relative flex items-start justify-between">
                  <span className="font-mono text-xs tracking-widest opacity-70">
                    {String(i + 1).padStart(2, "0")}
                  </span>
                  {(() => {
                    const Icon = getSubjectTheme(subject.slug).Icon;
                    return <Icon className="size-7 text-primary" strokeWidth={1.6} aria-hidden />;
                  })()}
                  <span className="font-mono text-xs opacity-70">
                    {subject.courses.length} cours
                  </span>
                </div>

                {subject.slug === "philo" && (
                  <PhyloCountdown className="relative border-t border-line pt-2" />
                )}

                <div className="relative flex items-end justify-between gap-4">
                  <div className="min-w-0 text-xl sm:text-2xl font-semibold leading-tight">
                    {subject.short}
                  </div>
                  <div className="hidden sm:block text-xs text-primary shrink-0">
                    Ouvrir →
                  </div>
                </div>
              </Link>
            ))}
          </div>
        )}
      </section>

      <div className="relative z-10 max-w-6xl mx-auto px-6 py-8">
        {founder && <FounderAlerts />}
        {founder && <FounderInbox />}
        {founder && <TeachersPanel />}
        {audience !== "autre" && !teacher && <ChatBox founder={founder} />}
        {audience !== "autre" && !teacher && <PrivateChat founder={founder} />}
        <SuggestionBox founder={founder} />
        <div className="mt-8 flex flex-wrap gap-3 items-center justify-between border-t border-border pt-4">
          <div className="font-mono text-xs uppercase tracking-[0.25em] text-mute">
            {founder ? "Gaby Rapide · Accès fondateur · chiffré" : "Gaby Rapide · Terminale 2026"}
          </div>
          <Button variant="ghost"
            onClick={lock}
            className="font-mono text-xs uppercase tracking-[0.2em] text-mute hover:text-acc"
          >
            Verrouiller ↺
          </Button>
        </div>
      </div>
    </main>
  );
}
