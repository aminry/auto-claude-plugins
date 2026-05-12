# I Built Two Plugins That Let Claude Code Improve Itself. Here's What Happened.

*How a Karpathy [tweet](https://x.com/karpathy/status/2030371219518931079) turned into a workflow I can't live without.*

---

Fix a bug. Run the tests. Break something else. Fix that. Run the tests again. Repeat until you're tired or it's Friday, whichever comes first.

Every developer knows this loop. It's the least fun part of the job, and it eats entire afternoons.

I saw [Karpathy's autoresearch post](https://x.com/karpathy/status/2030371219518931079) and had a simple thought: what if AI could run this loop *for* me? Not just generate code once and hope for the best. Actually try a fix, measure whether it helped, keep it if it did, revert it if it didn't, and move on to the next problem. On repeat. Until the code is actually good.

That idea became two open-source Claude Code plugins: **auto-improve** (fix quality issues) and **auto-develop** (add new features). Both work the same way: define what "good" looks like, let AI iterate toward it, only keep what works.

I use them for basically all my feature work now. Here's how they work.

---

## The Core Idea

Think about how you'd improve something if you had infinite patience.

You'd define exactly what "good" looks like. Not vaguely ("make it better") but specifically, with a checklist. Then you'd evaluate your work against that checklist, find the biggest problem, fix just that one thing, and check again. If the fix helped, keep it. If it made things worse, undo it. Then move to the next biggest problem.

That's what these plugins do. The trick is that "what good looks like" gets turned into something concrete and measurable, so an AI can actually evaluate it without you being in the loop.

![The Iterative Loop: Evaluate → Analyze failures → Fix → Re-evaluate → Accept or Reject → Repeat](https://excalidraw.com/#json=WQoWWCINnomOzTRZr1P4-,wSCLB29ceX-b4ZVngnz7gw)

There are two flavors:

**auto-improve** is the red pen. You have a feature that works but the output quality is rough. The plugin builds a rubric (40 to 100 yes/no questions about what good output looks like), scores your current output against it, and iteratively fixes the biggest problems. Think of it like a spell checker for your feature's behavior, except instead of just flagging issues, it actually fixes them and proves the fix worked.

**auto-develop** is the builder. You have a feature that works but it's missing capabilities. The plugin builds a prioritized specification (10 to 20 new things your feature should do, organized by dependency), then implements them one at a time. Each new capability gets verified on its own, and the entire existing feature gets regression-tested to make sure nothing broke.

You still decide what "good" looks like. You still review the changes. The plugins handle the tedious part: the actual iteration.

---

## auto-improve: The Quality Fixer

Let's walk through a real example. Say you built a CLI tool that converts Markdown resumes into PDFs. It works, you feed it a `.md` file and get a PDF out. But the output has problems. Spacing between sections is inconsistent. Long job titles overflow their containers. If someone lists twenty skills instead of six, the layout falls apart. Resumes with missing sections (no education? no summary?) leave awkward blank gaps. Dates are formatted three different ways in the same document.

You could spend all day opening PDFs, squinting at margins, and tweaking the rendering code one case at a time. Or you could let the plugin handle it:

```bash
/auto-improve:create pdf-render
```

Here's what happens. The plugin scans your project, figures out your tech stack (Node.js, Python, Go, it handles all the major ones), and analyzes the feature. Then it asks you a round of questions about what "good" means for this feature. The nice part: it pre-fills recommended answers based on what it found in your code. Questions like "What does good output look like?" "What are the most common problems?" "How do we run this in isolation?" You're mostly just confirming what it already figured out.

From your answers, it generates two things:

1. **An evaluation rubric** with 40 to 100 yes/no questions, grouped by quality dimension. Things like "Is the spacing between sections consistent?" "Do long job titles wrap cleanly without overflow?" "Are dates formatted uniformly throughout the document?" "Does a resume with a missing education section still render without blank gaps?" Each question is designed to be objectively answerable from the actual PDF output. No subjective judgment needed.

2. **An auto-improve skill** that runs the entire iterative loop hands-free, without needing you to confirm anything along the way.

Now you kick it off:

```bash
/auto-improve-pdf-render
```

And it goes to work. It runs your resume generator against a set of test Markdown files (short resumes, long ones, ones with missing sections, edge cases), evaluates every PDF against the rubric, and calculates a score. Let's say you're starting at 58%.

The plugin looks at all the failing rubric questions across all test resumes and clusters them. Maybe 12 questions are failing because of inconsistent section spacing. That's the biggest cluster, so it makes one focused code change to fix that specific issue. Then it re-evaluates *everything* against the full rubric.

Here's the key decision: if the mean score went up and the floor score (your worst-performing test resume) didn't drop, the change gets committed. If not, it gets reverted completely. No partial fixes. No "well it helped the short resumes but broke the long ones." The bar is: things got better overall and nothing got worse.

Over six iterations, your score goes from 58% to 91%. Each iteration tackled a different problem: section spacing, long-content overflow, missing-section handling, date formatting, bullet point alignment, font size consistency. You were drinking coffee the whole time.

![auto-improve score progression: 58% to 91% over 6 iterations](https://excalidraw.com/#json=WGkPEYYejx9jA68bB_GBT,lWSz8pOvXgcwcQzEpAy0vQ)

One more thing. After running a cycle, you might realize the rubric itself has blind spots. Maybe your PDFs have a page break issue (content getting cut in half between pages) but no rubric question catches it. That's what `evolve-rubric` is for:

```bash
/auto-improve:evolve-rubric pdf-render
```

It analyzes the gap between real issues and what the rubric covers, proposes new questions to close those gaps, and updates the rubric. Then you can run another improvement cycle with the smarter rubric. The rubric itself gets better over time.

![Meanwhile, the developer drinks coffee while auto-improve runs](https://excalidraw.com/#json=SBQjLSaow97nnsenfOyfa,bvGG-We8cstl20__IRsCmw)

---

## auto-develop: The Feature Builder

Same resume generator. The PDF rendering is clean now (thanks, auto-improve), but the tool is still bare-bones. It does exactly one thing: Markdown in, PDF out. You want multiple template themes, ATS-friendly plain text export, custom CSS support, maybe a `--watch` flag for live preview while you edit. You've got a whole wishlist of things that would make this tool genuinely useful.

Here's where auto-develop comes in. You can tell it exactly what you want:

```bash
/auto-develop:create resume-gen I want multiple template themes, ATS-friendly plain text export, custom CSS support, and a --watch flag for live preview
```

The plugin takes your wishlist as a starting point, scans your codebase to understand what exists, and proposes a full capability spec organized across three tiers. Think of it like a skill tree in a video game, where you need to unlock foundational abilities before the advanced ones become available:

**Tier 1 (Foundation):** Things that need to exist before anything else can work. In our case, that's a template engine abstraction and a pluggable output formatter. These aren't features your users see directly, but without them, themes and export formats would each be a one-off hack instead of a clean extension.

**Tier 2 (Core):** The features your users actually want. Multiple themes, plain text export, custom CSS injection, JSON resume format import. Each one depends on the foundation being in place.

**Tier 3 (Enhancement):** Polish and advanced features. Live preview with `--watch`, LinkedIn profile scraping, a built-in theme gallery command. Nice to have, not essential.

![Capability skill tree: foundation capabilities unlock core features, which unlock enhancements](https://excalidraw.com/#json=47x7uDr-kYEJpPy80jE7y,m0t_L2Ozd2bXauAHV92eyA)

Each capability comes with dependencies (multiple themes depend on the template engine), acceptance criteria (specific testable conditions that prove the capability works), and a list of files likely involved. You review the full list, accept or reject proposals, re-tier anything that's misplaced, and add capabilities the plugin missed. Then it generates your spec and your auto-develop skill:

```bash
/auto-develop-resume-gen
```

Here's the key difference from auto-improve: auto-develop *plans before it codes*. For each capability, it writes an implementation plan document, reads the relevant parts of your codebase, follows your existing patterns, then implements. This matters because new features are more complex than quality fixes. You don't want the AI just throwing code at the wall.

![The auto-develop Loop: Select capability → Plan → Implement → Verify → Regression check → Accept or Reject → Repeat](https://excalidraw.com/#json=PM2rSWgeHgN3-QSoUJilX,51XGjGFz38jjx_c9qxqyNg)

After implementing, there are two checks. First: does the new capability actually work? The plugin runs the acceptance criteria specific to that capability. Second: did it break anything that was already working? It runs the full regression suite, which covers both the original baseline (your "Markdown in, PDF out" flow still works) and every previously-accepted capability.

If both pass, the capability gets committed and added to the regression suite. If either fails, the code gets reverted completely. No broken code sneaks through. And here's the clever part: the regression suite *grows* with every successful implementation. By the time you're on capability #8, the safety net is comprehensive.

After a run, let's say 8 of your 10 targeted capabilities got implemented successfully. Two were marked as too complex for a single iteration (maybe the LinkedIn scraper needed network mocking infrastructure that didn't exist yet). Zero regressions. Your simple resume CLI now has themes, plain text export, custom CSS, and a watch mode.

And just like auto-improve has `evolve-rubric`, auto-develop has `evolve-spec`:

```bash
/auto-develop:evolve-spec resume-gen
```

This looks at what was built and discovers follow-on capabilities that are now possible. Now that you have a template engine and multiple themes, maybe a "create your own theme" scaffolding command makes sense. Or now that you have plain text export, an "email-friendly inline HTML" export is low-hanging fruit. The spec grows organically with your codebase.

---

## When to Use Which

The short version: if your feature works but the output is bad, use **auto-improve**. If your feature works but it needs to do more things, use **auto-develop**.

They also work great together. Use **auto-develop** to build out new capabilities, then use **auto-improve** to polish the quality of what was built. Build it, then make it good.

![When to use auto-improve vs auto-develop](https://excalidraw.com/#json=YnOugF6McCl0JkQjRGCqP,jNemJU-hiVHwuPND_lrWGQ)

---

## Get Started

Install both plugins from the Claude Code marketplace:

```bash
/plugin marketplace add aminry/auto-claude-plugins
/plugin install auto-improve@auto-claude-plugins
/plugin install auto-develop@auto-claude-plugins
/reload-plugins
```

**For teams**, you can add them to your project's settings.json so everyone gets them automatically:

```json
{
  "extraKnownMarketplaces": {
    "auto-claude-plugins": {
      "source": {
        "source": "github",
        "repo": "aminry/auto-claude-plugins"
      }
    }
  },
  "enabledPlugins": {
    "auto-improve@auto-claude-plugins": true,
    "auto-develop@auto-claude-plugins": true
  }
}
```

**Your first run:**

1. Pick a feature you want to improve or extend
2. Run `/auto-improve:create my-feature` or `/auto-develop:create my-feature`
3. Answer the interview questions (most will have pre-filled recommendations)
4. Review the generated rubric or spec, tweak anything that doesn't match your expectations
5. Run the generated skill and let it work

**Tips for getting the best results:**

- Start with a feature that has clear, measurable quality criteria. "Well-formatted PDF output" is a great candidate. "Nice code" is too vague.
- Be specific during the interview phase. The more precise your definition of "good," the better the results.
- Use `evolve-rubric` or `evolve-spec` after your first cycle to catch what the initial setup missed.

**Works with:** Node.js/TypeScript, Python, Go, Rust, Ruby, and Java/Kotlin. The plugins auto-detect your stack and adapt accordingly.

---

## What I Learned

The AI isn't brilliant. It sometimes makes dumb fixes that get correctly reverted. What makes this work isn't intelligence. It's the loop. Define "good." Measure against it. Only keep what actually improves things. The AI doesn't need to be perfect, it just needs a way to measure progress.

That turns out to be the real insight. AI without evaluation criteria is a coin flip. AI with a rubric and a feedback loop is a process. It's the difference between asking someone to "make this better" (a vague hope) and giving them a checklist of exactly what "better" means (something they can actually execute on).

I use these plugins for basically all my feature work now. auto-develop to build out capabilities, auto-improve to polish quality. The combination handles maybe 80% of the tedious iteration that used to eat my afternoons. And the stuff it can't handle (the truly creative architectural decisions, the "should we even build this" questions) is exactly the stuff I *want* to spend my time on.

The best AI tools don't try to nail it in one shot. They try, measure, learn, and try again. Just like we do. But they don't get tired on iteration seven. They don't forget what they already tried. They don't lose focus because Slack pinged.

Both plugins are open source. Try them, break them, tell me what's wrong with them. That's how they'll get better (iteratively, of course).

**GitHub:** [github.com/aminry/auto-claude-plugins](https://github.com/aminry/auto-claude-plugins)
