---
description: Generate HTML presentation slideshows (PPT-style) from content or topics. Creates self-contained, interactive HTML files with navigation, animations, dark theme, and responsive scaling. Use when the user asks to create a presentation, slideshow, PPT, or slide deck in HTML format.
---

# HTML PPT Generator

Generate beautiful, self-contained HTML presentation slideshows. The output is a single `.html` file — no build tools, no dependencies, no CDN.

## Workflow

### 1. Gather Requirements

Ask only what's needed:
- **Topic / content**: What is the presentation about? Source material (markdown, docs, codebase)?
- **Audience & tone**: Technical team, management, clients? Formal or casual?
- **Slide count**: Rough target (default 8-12)
- **Attribution**: Author name (for cover + ending slide)
- **Theme preference**: Color scheme (default: dark theme, blue-purple accent)

### 2. Declare Design System

Before writing code, state:

```markdown
Design Decisions:
- Canvas: 1920x1080 (16:9), auto-scaled via JS transform
- Color: [dark/light], [primary accent], [secondary accent]
- Font: [heading font], [body font], [code font]
- Navigation: [bottom bar / side arrows / dots]
- Transitions: [slide direction, duration, easing]
```

### 3. Build the HTML File

Use the slide engine template below. Key conventions:

#### Canvas & Scaling

```html
<style>
  *,*::before,*::after{margin:0;padding:0;box-sizing:border-box}
  :root{
    --bg-dark:#000000;--bg-slide:#000000;
    --accent-blue:#6B9FFF;--accent-purple:#A78BFA;--accent-violet:#C4B5FD;
    --text-primary:#F5F5F7;--text-secondary:#C0C0C4;--text-dim:#8E8E93;
    --card-bg:rgba(255,255,255,0.05);--card-border:rgba(255,255,255,0.2);
  }
  html,body{width:100%;height:100%;overflow:hidden;background:var(--bg-dark);
    font-family:-apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,Arial,sans-serif;
    color:var(--text-primary)}
  #stage{
    position:relative;width:100vw;height:100vw;max-height:100vh;aspect-ratio:16/9;
    display:flex;align-items:center;justify-content:center;
  }
  #slideshow{
    width:1920px;height:1080px;position:relative;overflow:hidden;
    background:var(--bg-slide);
  }
</style>
```

#### Slides

Each slide is a `<div class="slide">` with `data-screen-label="NN Title"` (1-indexed, zero-padded):

```html
<div class="slide active" data-screen-label="01 Cover">
  <!-- content -->
</div>
<div class="slide" data-screen-label="02 Agenda">
  <!-- content -->
</div>
```

CSS for transitions:

```css
.slide{
  position:absolute;inset:0;
  display:flex;flex-direction:column;padding:80px 100px;
  opacity:0;transform:translateX(60px);
  transition:opacity 0.5s cubic-bezier(0.4,0,0.2,1),
             transform 0.5s cubic-bezier(0.4,0,0.2,1);
  pointer-events:none;
}
.slide.active{opacity:1;transform:translateX(0);pointer-events:auto}
.slide.exit-left{opacity:0;transform:translateX(-60px)}
```

#### Bottom Navigation Bar

Place a capsule bar at the bottom with prev/dot/page-num/fs/next:

```html
<div id="nav-bar">
  <button class="nav-arrow" id="prev-btn">&#x25C0;</button>
  <div id="dots"></div>
  <span class="page-num" id="pageNum"></span>
  <div class="nav-sep"></div>
  <button id="fs-btn">&#x26F6;</button>
  <button class="nav-arrow" id="next-btn">&#x25B6;</button>
</div>
```

#### JS Slide Engine

```javascript
(function(){
  const slides = document.querySelectorAll('.slide');
  const dotsContainer = document.getElementById('dots');
  const pageNum = document.getElementById('pageNum');
  let current = 0;
  const total = slides.length;
  let transitioning = false;

  // Build dots
  for(let i = 0; i < total; i++){
    const d = document.createElement('div');
    d.className = 'dot' + (i === 0 ? ' active' : '');
    d.onclick = () => goTo(i);
    dotsContainer.appendChild(d);
  }

  // Arrow buttons
  document.getElementById('prev-btn').onclick = () => navigate(-1);
  document.getElementById('next-btn').onclick = () => navigate(1);

  // Fullscreen
  const fsBtn = document.getElementById('fs-btn');
  fsBtn.onclick = () => {
    if(!document.fullscreenElement)
      document.documentElement.requestFullscreen().catch(()=>{});
    else
      document.exitFullscreen().catch(()=>{});
  };
  document.addEventListener('fullscreenchange', () => {
    fsBtn.innerHTML = document.fullscreenElement ? '&#x2715;' : '&#x26F6;';
    setTimeout(scale, 100);
  });

  function update(){
    slides.forEach((s, i) => {
      s.classList.remove('active', 'exit-left');
      if(i === current) s.classList.add('active');
      else if(i < current) s.classList.add('exit-left');
    });
    document.querySelectorAll('.dot').forEach((d, i) => {
      d.classList.toggle('active', i === current);
    });
    pageNum.textContent = String(current+1).padStart(2,'0') + ' / ' + String(total).padStart(2,'0');
    try { localStorage.setItem('slide_pos', current); } catch(e){}
  }

  function navigate(dir){
    if(transitioning) return;
    const next = current + dir;
    if(next < 0 || next >= total) return;
    transitioning = true;
    current = next;
    update();
    setTimeout(() => { transitioning = false; }, 520);
  }

  function goTo(i){
    if(transitioning || i === current) return;
    transitioning = true;
    current = i;
    update();
    setTimeout(() => { transitioning = false; }, 520);
  }

  // Keyboard
  document.addEventListener('keydown', e => {
    if(e.key === 'ArrowRight' || e.key === ' '){ e.preventDefault(); navigate(1); }
    else if(e.key === 'ArrowLeft'){ e.preventDefault(); navigate(-1); }
    else if(e.key === 'f' || e.key === 'F'){ fsBtn.click(); }
  });

  // Mouse wheel
  let wheelAccum = 0;
  document.addEventListener('wheel', e => {
    e.preventDefault();
    wheelAccum += e.deltaY;
    if(Math.abs(wheelAccum) >= 80){
      if(wheelAccum > 0) navigate(1); else navigate(-1);
      wheelAccum = 0;
    }
    clearTimeout(window._wheelTimer);
    window._wheelTimer = setTimeout(() => { wheelAccum = 0; }, 300);
  }, { passive: false });

  // Click left/right third of stage
  document.getElementById('stage').addEventListener('click', e => {
    if(e.target.closest('#nav-bar')) return;
    const rect = e.currentTarget.getBoundingClientRect();
    const x = e.clientX - rect.left;
    const w = rect.width;
    if(x < w * 0.3) navigate(-1);
    else if(x > w * 0.7) navigate(1);
  });

  // Touch
  let touchStartX = 0;
  document.addEventListener('touchstart', e => { touchStartX = e.touches[0].clientX; }, { passive: true });
  document.addEventListener('touchend', e => {
    const dx = e.changedTouches[0].clientX - touchStartX;
    if(Math.abs(dx) > 50){ dx > 0 ? navigate(-1) : navigate(1); }
  }, { passive: true });

  // Restore position
  try {
    const saved = parseInt(localStorage.getItem('slide_pos'));
    if(!isNaN(saved) && saved > 0 && saved < total) current = saved;
  } catch(e){}

  // Scale to fit viewport
  function scale(){
    const ss = document.getElementById('slideshow');
    const sw = window.innerWidth;
    const sh = window.innerHeight;
    const s = Math.min(sw / 1920, sh / 1080);
    ss.style.transform = 'scale(' + s + ')';
    ss.style.transformOrigin = 'center center';
    const stage = document.getElementById('stage');
    stage.style.width = (1920 * s) + 'px';
    stage.style.height = (1080 * s) + 'px';
  }
  window.addEventListener('resize', scale);
  scale();
  update();
})();
```

### 4. Slide Content Patterns

Use these reusable component styles:

#### Cover Slide
```html
<div class="slide active" data-screen-label="01 Cover">
  <div class="cover-content">
    <div class="cover-tag">[Category / Tagline]</div>
    <h1>[Title]</h1>
    <div class="subtitle">[Subtitle]</div>
    <div class="accent-bar accent-bar-wide mt-32"></div>
    <p class="body-text mt-24">[Description]</p>
    <p class="small-text mt-32">[Date]</p>
    <p class="small-text mt-16" style="font-size:14px">© [Author]</p>
  </div>
</div>
```

#### Tool / Topic Slide (Two Column)
```html
<div class="slide" data-screen-label="NN Title">
  <div class="content-area">
    <span class="section-label">[Section Label]</span>
    <h2>[Title]</h2>
    <div class="accent-bar mt-24"></div>
    <div class="grid-2 mt-32">
      <div><!-- description, code, pros/cons --></div>
      <div><!-- card with diagram / table / architecture --></div>
    </div>
  </div>
</div>
```

#### Comparison Table Slide
```html
<div class="slide" data-screen-label="NN Comparison">
  <div class="content-area">
    <span class="section-label">Comparison</span>
    <h2>[Title]</h2>
    <div class="accent-bar mt-24"></div>
    <div class="table-wrap mt-24">
      <table>
        <thead><tr><th>Feature</th><th>A</th><th>B</th><th>C</th></tr></thead>
        <tbody>
          <tr><td class="text-dim">...</td><td><span class="badge badge-y">Yes</span></td>...</tr>
        </tbody>
      </table>
    </div>
  </div>
</div>
```

#### Ending Slide
```html
<div class="slide" data-screen-label="NN End">
  <div class="ending">
    <div class="cover-tag">Thank You</div>
    <h1 style="font-size:80px">Q & A</h1>
    <div class="accent-bar accent-bar-wide mt-32"></div>
    <p class="subtitle mt-24">[Title]</p>
    <p class="small-text mt-24">[Author] · [Year]</p>
  </div>
</div>
```

### 5. Verify

- [ ] No external CDN dependencies (fully self-contained)
- [ ] Keyboard ← → Space F all work
- [ ] Mouse wheel, click, touch all work
- [ ] Fullscreen toggles correctly
- [ ] localStorage persists position
- [ ] Scales correctly on resize
- [ ] No console errors
- [ ] Attribution on cover + ending

### 6. Save

Save as `[topic]_presentation.html` in the current working directory.
