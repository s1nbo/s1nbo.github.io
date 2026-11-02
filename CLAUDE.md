# Sinan Bove – personal website

Personal portfolio site for Sinan Bove (MSc Computer Science, TU Berlin; AI with a specialization in reinforcement learning).
Goal: be seen by recruiters, companies and research labs. Applying is on pause; the site does the talking.

This file is gitignored. It is working notes for Claude, not part of the site.

## Files and hosting

- `index.html` is the whole site: one self-contained file (HTML + CSS + JS). The only external request is Google Fonts.
- Hosted with GitHub Pages from the private repo `cv` (GitHub Pro). The site itself is public.
  - URL with the current repo name: `https://s1nbo.github.io/cv/`
  - Planned: rename the repo to `s1nbo.github.io` or add a custom domain, so the site serves at the root.
- Deploy: commit `index.html` to `main`; Pages (Settings → Pages → Deploy from a branch → main / root) updates in about a minute.
- Test locally by opening `index.html` in a browser. Force a hero scene with `?scene=ql`, `?scene=az` or `?scene=pg`.
- A private preview also exists as a claude.ai artifact; the artifact version has no `<!doctype>/<head>/<body>` wrapper. `index.html` adds that wrapper, a meta description, OG title/description and an inline SVG hexagon favicon.

## Design direction (agreed, keep it)

- Inspired by top architecture firms, not CS portfolios: BIG (sparse feed, fact sheets, step-by-step diagram sequences), MVRDV (condensed uppercase type, menu animation), ZHA (dark parametric hero).
- Visuals in the spirit of 3Blue1Brown: every image is a live, mathematically drawn animation in blue and gold on near-black.
- Near-black page; colour comes only from the animations. Single dark theme by choice.
- Clean, minimal text. Text appears only where needed or on click.

Tokens (`:root`):
- `--ink #05070b`, `--ink-2 #0a0e15`, `--ink-3 #10161f`, `--rule #161d29`, `--rule-2 #283245`
- `--chalk #e8ecf2` (text), `--fog #8a95ab`, `--dust #5a6680`
- `--blue #58c4dd`, `--gold #e3a857` (accent / highlight), `--teal #5cd0b3`
- Fonts: Archivo (display, condensed 62% width, uppercase headlines; header name at 125% width, weight 800), Instrument Sans (body), STIX Two Text italic (math, labels inside drawings).

## Page structure

1. **Intro**: black screen, "SINAN BOVE" centred, then it glides into the header top left. Plays on every load (hold ~1.7 s, glide 1.45 s). The page always opens at the top on refresh (scroll restoration off, in-page links do not change the URL hash).
2. **Header**: name top left; three equal grey bars top right (menu).
   - Menu animation copied from MVRDV: bars widen on hover; on open the middle bar spins away and the outer bars rotate into an X. Overlay fades in with blur, then each section name scales up from 0, one after another, with a gold underline growing from the centre on hover.
   - Menu items: Publications, Projects, Work, Education, Contact.
3. **Hero**: one live animation picked at random on each load (never the same twice in a row, stored in localStorage `sb-hero-scene`), plus:
   - About text bottom left, then a compact skills row.
   - Bottom right: HUD label (algorithm + progress), small sparkline, and a "→" skip button that cycles scenes. The user likes the skip button; keep it.
   - `heroSpace()` measures the free area between header and about text so diagrams never overlap the text (matters on short laptop screens).
4. **Publications** and **Projects**: kept as separate sections (more items will be added). Count shown next to the heading.
   - Feed tiles are square drawings only, no text, centred. The only text is a small italic label inside the drawing, bottom right: venue for publications, year for projects.
   - Hover: drawing animates and lights up (gold border).
   - Click: full-screen project sheet. Only an X to close (same X morph as the menu), no top-bar text, no "Next work", no figure captions, no "Scale" field. Facts: Context, Year, Typology, Status, Stack. Then summary, real links only, and a step-through diagram (arrows step through layers).
5. **Work**, 6. **Education**: rows with dates, role, note, location. Coursework in a smaller line.
7. **Contact**: large email with copy button, "CV available on request", GitHub and LinkedIn buttons.
8. **Footer**: "© 2026 Sinan Bove · Berlin" and "Last updated: October 2026".

## Hero scenes (all run real algorithms in the browser)

- `ql` **Q-learning**: tabular Q-learning on a hex grid; terrain height = learned value, arrows = greedy policy, gold agent walks the learned path; new map once solved.
- `az` **AlphaZero**: PUCT tree search on Connect Four (board scale factor `1.02` in `azRender`, enlarged 1 Nov 2026 on request) with random-rollout values; the tree grows radially around the board, gold traces the latest simulations, gold bars above the board show visit counts, most-visited move is played.
- `pg` **DQN learning Pong**: the classic deep RL benchmark, trained live in the browser from random weights on every load.
  Nothing pretrained, nothing scripted. This is the scene the user wanted: you watch the paddle and think "what would I do",
  same as `ql` and `az`.
  Agent: 7 inputs (ball x, y, ẋ, ẏ, paddle y, ball−paddle gap, predicted impact − paddle) → 20 tanh units → 3 Q-values
  (up / stay / down). Double DQN, Adam lr 0.01, 4000-step replay buffer, target net copied every 150 updates, 4-step returns,
  γ 0.95, ε annealed 1 → 0.05 in practice and 0.005 on screen. Reward +1 per return, −1 per miss; only a miss ends the episode.
  Left paddle is a scripted tracker with a speed cap and noise, so it misses too.
  Two environments: the court on screen plays on the real clock, a second env practises off-screen at ~330 steps/s and feeds the
  same net. On ten seeds: 79 % of balls returned in the first 25 s, 93 % once trained, 85 % return rate reached at a median of
  23 s. Loop: ≥ 85 % for two blocks → plays on 16 s → new seed resets the weights and the arc replays.
  Drawing: court with dashed centre line, gold agent paddle, chalk opponent, ball with a short blue trail; dashed blue line gives
  the ball's path with wall bounces plus a tick at the impact point (the reader sees the answer before the paddle moves); gold
  `returns` training curve under the court with the live ε; network panel on the right (under the court below 620 px) with edges
  drawn blue for positive weights and teal for negative, node fill = live activation, and Q bars per action, gold on the argmax.
  Costs ~0.3 ms/frame. Reduced motion: 2600 practice steps (~100 ms) then a still frame mid-rally.
  Tuning that mattered, do not undo it (measured, 8–12 seeds each):
  - Physics runs at 1/120 s sub-steps consumed from the real clock, decisions every 8 sub-steps (1/15 s). Fixed 1/30 s chunks
    made some frames step twice and others not at all, which was the stutter the user saw.
  - Pong renders at 60 fps (`frame()` picks the step from `hero.S.key`), and the hero accumulator keeps its remainder instead of
    being zeroed — zeroing it made the render interval jitter between 33 and 50 ms.
  - Episodes no longer end on a return. With termination at every contact the agent was paid for luck: it tracked the ball only
    39 % of the time and plateaued at 89 %. Continuing episodes + 4-step returns + γ 0.95 took it to 93–96 %.
  - The predicted impact point is an input because 60 % of the remaining misses were balls that bounced off a wall on the way in,
    which a 20-unit net cannot infer from velocity alone. The net sees the same line the drawing shows.
  - Drawn paddle positions are eased (`pyShow`/`oyShow`, ~26/s) so bang-bang control does not read as twitching; direction changes
    dropped from 26 % to 14 % of decisions after the horizon fix.
  - Tried and rejected: reward shaping on |ball − paddle| (collapsed to 17 %), bigger paddle, slower ball, finer decisions at
    1/40 s, 28 hidden units, lower ε floor — all within noise or worse.
- Rejected and removed (do not bring back): policy gradient paths, spatial prisoner's dilemma, gradient descent / loss landscape,
  diffusion / DDPM image sampler (the only raster scene, clashed with the line-drawn rest of the site; removed 1 Nov 2026),
  self-attention over a sentence — first as three heads cycling, then as a next-word prediction with p(next) bars; both correct and
  both unsatisfying to look at (removed 1 Nov 2026). Lesson: a hero scene needs a visible decision the reader can call first.
- Respect `prefers-reduced-motion`: scenes render a finished still frame.

## Content data

All feed items live in the `WORKS` object near the top of the main script (`publications: [...]`, `projects: [...]`).
Fields: `id, kind, icon, title, subtitle, place, year, venue, typology, scale (unused), status, stack, fig (unused), summary, links[], steps[]`.
- `links`: only add entries with a real `href`. Never show placeholder buttons.
- `kind` picks the drawing: `catan`, `clusters`, `pillars`, `stairs`, `phone`, `dice` (add new kinds in `VISUALS`, 3Blue1Brown style, labels only shown in the project sheet).
- `steps`: 3–4 steps; each reveals one more layer of the drawing.

Current content:
- **Publication**: EcoTUB @ SustainEval 2025 (KONVENS 2025). Task A: classify German sustainability report snippets into 20 DNK criteria. Ensemble of seven fine-tuned GBERT models, Optuna tuning, 1,050 back-translated snippets. 0.74 accuracy / macro F1 on the dev set; 1st in validation phase, 2nd in final test. First author, with Icondy Kiba-Gassaye, Sirak Tadesse, Lisa Raithel. Paper: https://aclanthology.org/2025.konvens-2.19.pdf, code: https://github.com/s1nbo/sustaineval2025. Test accuracy (0.58) deliberately not shown.
- **Project**: fastCatan, MSc thesis (2025–26, in progress): C++ engine, PyTorch agents (PPO, AlphaZero-style search, Q-learning), shared-parameter self-play on TU Berlin HPC; question: when do self-interested agents learn to cooperate. GitHub: https://github.com/s1nbo/fastCatan. No results shown yet (user's call).
- **Work**: SAP, Product Manager Intern, Nov 2022 – Mar 2024, Remote. Multimodal support agent (text + images) diagnosing machine defects, asks customers for missing details in a reasoning loop, drafts part orders; work equal to 70 full-time employees. ABAP CDS views + OData streaming S/4HANA finance data to SAP Analytics Cloud. Stack line: Python, ABAP, SQL.
- **Education**:
  - MSc Computer Science, TU Berlin, Oct 2024 – Dec 2026. Thesis and coursework sit in one block, same style as the BSc row (user's call, 2 Nov 2026); grade 1.0 pulled for now, not deleted from the record. Thesis: Reinforcement Learning in Stochastic Multi-Agent Environments. Coursework: Applications of Robotics and Autonomous Systems, Machine Learning and Data Management Systems, Symbolic Artificial Intelligence (German name was cut off as "…mbolische Künstliche Intelligenz"; confirm Symbolische vs Subsymbolische).
  - Visiting Student, University of Oxford, Jun – Aug 2026: Deep Unsupervised Learning, Computer Vision, Generative AI.
  - BSc Computer Science, TU Berlin, Oct 2020 – Jan 2025. Coursework only (thesis removed on purpose).
- **About**: "I work on artificial intelligence, specializing in reinforcement learning: building agents that learn from experience, from game-playing systems to multi-agent settings where they compete and cooperate. MSc Computer Science at TU Berlin, open to research and industry roles across AI."
- **Skills row**: Python, PyTorch, C++, JAX, Gymnasium, PettingZoo, Stable-Baselines3, Hugging Face, LLM agents, scikit-learn, Docker, AWS, SQL, SLURM / HPC, W&B. ("LLM agents" based on the SAP agent; confirm.)
- **Contact**: bovesinan@gmail.com, https://github.com/s1nbo, https://www.linkedin.com/in/sinbo/

## Rules from the user

- Keep it minimal. No clutter; text only where needed or on click.
- Never leave placeholders (`[...]`, dashed buttons) on the live page. Leave a field out instead.
- No results or numbers that do not exist yet. fastCatan results come later.
- The CV is not public. Keep "CV available on request"; no CV download link, no phone number, no address.
- Do not add skills he has not used (e.g. TensorFlow, Kubernetes, Azure only if he confirms).
- Removed on purpose, do not re-add: Voidworks and its apps (Masswerk, CHAON, Siedler), BSc thesis project and line, policy gradient / prisoner's dilemma / gradient descent scenes.
- Keep Publications and Projects separate.
- Answers to the user: short and direct.

## Open to-dos

- Add new projects as they are finished (one real visual per project: plot, training curve or GIF).
- OG preview image (`og:image`, e.g. a hero still) before sharing on LinkedIn.
- Google Scholar profile and link.
- Optional: Honours line (SustainEval 2nd place, VW X-Starters top 10 Germany 2018, Oxford grades A / first class), short News strip, languages (German native, English TOEFL 111), "outside work" line (architecture, basketball, financial markets).
- Privacy-friendly analytics (GoatCounter or Plausible).
- Rename repo to `s1nbo.github.io` or add a custom domain.
- Update "Last updated" when content changes (now November 2026).

## Technical notes

- No build step, no frameworks. Keep everything in `index.html`.
- Canvas scenes share one `Vis` driver; the hero is throttled to ~30 fps and pauses when off-screen or when a project sheet is open.
- Avoid `alert/confirm/prompt`. Copy buttons fall back to selecting the text.
- Check layouts at 390×844 (phone), 1280×720 (small laptop) and 1440×900; the page must never scroll horizontally.
- Headless check: Playwright + Chromium, screenshot each scene with `?scene=…`, and confirm no console errors.
