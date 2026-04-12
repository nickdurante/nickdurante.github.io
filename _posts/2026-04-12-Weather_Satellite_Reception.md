---
title: "Weather Satellite Reception"
categories:
  - Satellite
  - SDR
comments: true
---

This project has been living rent-free in my head for a long time.
Finally pulling actual data out of the sky? Pure, unadulterated satisfaction.

The hardware I used is:

* **V-dipole antenna**: 54cm, angled at 120° 
* **Low pass SAW filter**: To keep the RF noise floor from ruining my life
* **LNA**: 20dB gain, 50MHz - 6GHz (overkill? maybe.)
* **RTL-SDR v3**

<p align="center">
  <img src="/assets/images/satellite_reception.jpg" width="600" alt="Setup">
</p>

For the capturing the signal and actually decoding the bits into something human-readable I used [Satdump](https://www.satdump.org). 


<p align="center">
  <img src="/assets/images/satdump.png" width="600" alt="SatDump">
</p>

Special thanks to **Naji** for the help to **Jacopo Cassinis** for his incredible work over at [A-Centauri](https://www.a-centauri.com/).

In the section below, I’ll be dumping some of the captures whenever the pass timing and the weather gods align with my free time.


# Gallery


<style>
  .gallery-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
    gap: 15px;
    padding: 20px 0;
  }

  .gallery-item {
    display: block;
    overflow: hidden;
    border-radius: 8px;
    box-shadow: 0 4px 8px rgba(0,0,0,0.1);
    transition: transform 0.2s ease-in-out;
    cursor: pointer;
  }

  .gallery-item:hover {
    transform: scale(1.02);
  }

  .gallery-item img {
    width: 100%;
    height: 250px;
    object-fit: cover;
    display: block;
  }

  #lightbox-overlay {
    display: none;
    position: fixed;
    z-index: 9999;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.9);
    justify-content: center;
    align-items: center;
    cursor: zoom-out;
  }

  #lightbox-overlay img {
    max-width: 90%;
    max-height: 85%;
    border: 4px solid white;
    border-radius: 4px;
    cursor: default;
  }

  .close-btn {
    position: absolute;
    top: 20px;
    right: 30px;
    color: white;
    font-size: 40px;
    font-weight: bold;
    cursor: pointer;
    line-height: 1;
  }

  #lightbox-caption {
    position: absolute;
    bottom: 20px;
    color: white;
    font-size: 14px;
    background: rgba(0, 0, 0, 0.5);
    padding: 6px 14px;
    border-radius: 4px;
    letter-spacing: 0.03em;
  }

  .nav-btn {
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
    color: white;
    font-size: 50px;
    font-weight: bold;
    cursor: pointer;
    user-select: none;
    padding: 10px 20px;
    background: rgba(0, 0, 0, 0.3);
    border-radius: 4px;
    line-height: 1;
    transition: background 0.2s;
  }

  .nav-btn:hover {
    background: rgba(0, 0, 0, 0.7);
  }

  #nav-prev {
    left: 20px;
  }

  #nav-next {
    right: 20px;
  }
</style>

<div class="gallery-grid">
  {% for file in site.static_files %}
    {% if file.path contains '/assets/images/satellites/' %}
      {% if file.extname == '.jpg' or file.extname == '.jpeg' or file.extname == '.png' or file.extname == '.webp' %}
        <a href="javascript:void(0)" class="gallery-item"
           onclick="openLightbox('{{ file.path | relative_url }}', '{{ file.name }}', '{{ file.modified_time | date: "%Y-%m-%d %H:%M" }}')">
          <img src="{{ file.path | relative_url }}" alt="Gallery Image" loading="lazy">
        </a>
      {% endif %}
    {% endif %}
  {% endfor %}
</div>

<div id="lightbox-overlay">
  <span class="close-btn" onclick="closeLightbox()">&times;</span>
  <span class="nav-btn" id="nav-prev" onclick="navigateLightbox(-1)">&#10094;</span>
  <img id="lightbox-img" src="" alt="Enlarged view">
  <span class="nav-btn" id="nav-next" onclick="navigateLightbox(1)">&#10095;</span>
  <div id="lightbox-caption">
    <span id="lightbox-time"></span>
  </div>
</div>

<script>
  document.body.appendChild(document.getElementById('lightbox-overlay'));

  // Build gallery index from all gallery items at load time
  const galleryItems = Array.from(document.querySelectorAll('.gallery-item')).map(el => ({
    src: el.getAttribute('onclick').match(/'([^']+)'/)[1],
    filename: el.getAttribute('onclick').match(/'([^']+)'/g)[1].replace(/'/g, ''),
    fallback: el.getAttribute('onclick').match(/'([^']+)'/g)[2].replace(/'/g, '')
  }));

  let currentIndex = 0;

  function parseTimestamp(filename) {
    const patterns = [
      /(\d{4})-(\d{2})-(\d{2})_(\d{2})-(\d{2})(.*)/,
      /(\d{4})-(\d{2})-(\d{2})_(\d{2})(\d{2})(.*)/,
      /(\d{4})(\d{2})(\d{2})_(\d{2})(\d{2})(\d{2})(.*)/,
      /(\d{4})(\d{2})(\d{2})_(\d{2})(\d{2})(.*)/,
    ];

    for (const pattern of patterns) {
      const m = filename.match(pattern);
      if (m) {
        const year  = m[1];
        const month = m[2];
        const day   = m[3];
        const hour  = m[4];
        const min   = m[5];
        const remainder = m[m.length - 1]
          .replace(/\.[^.]+$/, '')
          .replace(/^[_\s]+/, '')
          .replace(/[_]+/g, ' ')
          .trim();
        return {
          time: `${year}-${month}-${day} ${hour}:${min}`,
          label: remainder || null
        };
      }
    }
    return null;
  }

  function showImage(index) {
    const item = galleryItems[index];
    document.getElementById('lightbox-img').src = item.src;

    const parsed = parseTimestamp(item.filename);
    let timeStr;
    if (parsed) {
      timeStr = parsed.time;
      if (parsed.label) timeStr += ' — ' + parsed.label;
      
    }
    document.getElementById('lightbox-time').textContent = '🕐 ' + timeStr;

    // Hide arrows at the ends
    document.getElementById('nav-prev').style.visibility = index === 0 ? 'hidden' : 'visible';
    document.getElementById('nav-next').style.visibility = index === galleryItems.length - 1 ? 'hidden' : 'visible';
  }

  function openLightbox(src, filename, fallbackTime) {
    currentIndex = galleryItems.findIndex(item => item.src === src);
    showImage(currentIndex);
    document.getElementById('lightbox-overlay').style.display = 'flex';
    document.body.style.overflow = 'hidden';

    const header = document.querySelector('.masthead')
      || document.querySelector('.site-header')
      || document.querySelector('header.greedy-nav')
      || document.querySelector('header');
    if (header) header.style.display = 'none';
  }

  function navigateLightbox(direction) {
    const newIndex = currentIndex + direction;
    if (newIndex >= 0 && newIndex < galleryItems.length) {
      currentIndex = newIndex;
      showImage(currentIndex);
    }
  }

  function closeLightbox() {
    document.getElementById('lightbox-overlay').style.display = 'none';
    document.body.style.overflow = 'auto';

    const header = document.querySelector('.masthead')
      || document.querySelector('.site-header')
      || document.querySelector('header.greedy-nav')
      || document.querySelector('header');
    if (header) header.style.display = '';
  }

  document.addEventListener('keydown', function(e) {
    if (e.key === "Escape") closeLightbox();
    if (e.key === "ArrowLeft")  navigateLightbox(-1);
    if (e.key === "ArrowRight") navigateLightbox(1);
  });

  // Prevent clicks on nav buttons and image from closing the lightbox
  document.getElementById('lightbox-overlay').addEventListener('click', function(e) {
    if (e.target === this) closeLightbox();
  });
</script>