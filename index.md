---
layout: default
title: I'm Anika Marathe
hide: true
---

### Me and Team

Hi! My name is Anika.

| Role | Name | Repo Location | Stream | Repo Name |
|---|---|---|---|---|
| Scrum Master | John | github.com/jm1021/student | upstream (OCS fork) | |
| Scrummer | Torin | github.com/torin/student | downstream (fork) | |
| Scrummer | Avantika | github.com/avantika/student | downstream (fork) | |
| Scrummer | Aadit | github.com/aaadit/student | downstream (fork) | |

## Links to Learning

### Development Environment

> Coding starts with tools, explore these tools and procedures with a click.

<a href="https://github.com/Open-Coding-Society/student">
    <img src="https://img.shields.io/badge/GitHub-181717?logo=github&logoColor=white" alt="GitHub">
</a>
<a href="https://open-coding-society.github.io/student">
    <img src="https://img.shields.io/badge/GitHub%20Pages-327FC7?logo=github&logoColor=white" alt="GitHub Pages">
</a>
<a href="https://kasm.opencodingsociety.com/"
   class="button small"
   style="background-color: #d34bafff; border: 2px solid black; border-radius: 8px;">
    KASM
</a>

<a href="https://vscode.dev/" class="button small" style="background-color: #d38a4bff; padding: 10px 20px; border-radius: 8px; text-decoration: none; transition: all 0.3s ease;">
    <span style="color: #1295b3ff; font-weight: bold;">VSCODE</span>
</a>

<style>
/* ------------------- LIGHT MODE STYLES (DEFAULT) ------------------- */
body.light-mode {
    background: linear-gradient(to bottom right, #ffd6e8, #ffe6f0);
    font-family: 'Comic Sans MS', cursive, sans-serif;
    color: #660033; /* default text color */
    background-image: url("images/cloud.jpg"); /* cloudy day */
    background-repeat: no-repeat;
    background-size: cover;
    background-position: center center;
    background-attachment: fixed;
    transition: background-color 0.5s, color 0.5s; /* Smooth transition */
}

/* Light mode specific text and link colors */
.light-mode h3 {
    color: #ff3399;
    text-shadow: 1px 1px 2px #ffa3c6;
}

.light-mode a {
    color: #cc0066;
    text-decoration: none;
}

.light-mode a:hover {
    color: #ff3399;
    text-decoration: underline;
}

.light-mode table {
    background-color: #ffd6e8;
    color: #660033; /* Text color inside table */
    border-radius: 12px;
    padding: 10px;
    box-shadow: 0 4px 8px rgba(255, 102, 179, 0.4);
}

.light-mode th, .light-mode td {
    color: #660033;
}


/* ------------------- DARK MODE STYLES ------------------- */
body.dark-mode {
    background-color: #121212; /* A dark, not quite black, background */
    font-family: 'Comic Sans MS', cursive, sans-serif;
    color: #e0e0e0; /* A soft white for text */
    background-image: none; /* No wallpaper in dark mode */
    transition: background-color 0.5s, color 0.5s; /* Smooth transition */
}

/* Dark mode specific text and link colors */
.dark-mode h3 {
    color: #bb86fc; /* A nice pastel purple for dark mode headers */
    text-shadow: none;
}

.dark-mode a {
    color: #03dac6; /* A bright teal for links */
    text-decoration: none;
}

.dark-mode a:hover {
    color: #3700b3;
    text-decoration: underline;
}

.dark-mode table {
    background-color: #1e1e1e; /* Darker background for table */
    color: #e0e0e0; /* Text color inside table */
    border: 1px solid #bb86fc;
    border-radius: 12px;
    padding: 10px;
    box-shadow: none;
}

.dark-mode th, .dark-mode td {
    color: #e0e0e0;
}


/* ------------------- SHARED STYLES (FOR BOTH MODES) ------------------- */
.button.small {
    border-radius: 12px;
    transition: all 0.4s ease;
}

.button.small:hover {
    transform: scale(1.08) rotate(1deg);
    box-shadow: 0 0 12px #ff99cc, 0 0 20px #ffb3d9;
}

.theme-btn {
    background: linear-gradient(135deg, #ff99cc, #ff66b3);
    color: #fff;
    font-weight: bold;
    border: none;
    border-radius: 12px;
    padding: 10px 20px;
    margin: 5px;
    cursor: pointer;
    box-shadow: 0 4px 8px rgba(0,0,0,0.2);
}

.theme-btn:hover {
    transform: scale(1.08) rotate(-2deg);
    box-shadow: 0px 0px 12px #ff99cc, 0px 0px 20px #ff66b3;
}
</style>

### Class Progress

<a href="{{site.baseurl}}/snake" class="button small" style="background-color: #6b4bd3ff">
    Snake Game
</a>

<a href="{{site.baseurl}}/turtle" class="button small" style="background-color: #2A7DB1">
    <span style="color: #000000">Turtle</span>
</a>

<button onclick="toggleTheme()" class="theme-btn">🌗 Toggle Theme</button>

<br>

<!-- Contact Section -->
### Get in Touch

> Feel free to reach out if you'd like to collaborate or learn more about our work.

<p style="color: #2A7DB1;">Open Coding Society: <a href="https://opencodingsociety.com" style="color: #2A7DB1; text-decoration: underline;">Socials</a></p>

<!-- ======================= JAVASCRIPT FOR THEME TOGGLE ======================= -->
<script>
    // Function to toggle between light and dark mode
    function toggleTheme() {
        const body = document.body;
        // Check if dark mode is currently active
        const isDarkMode = body.classList.contains('dark-mode');
        // Toggle the class and save the preference
        if (isDarkMode) {
            body.classList.replace('dark-mode', 'light-mode');
            localStorage.setItem('theme', 'light-mode');
        } else {
            body.classList.replace('light-mode', 'dark-mode');
            localStorage.setItem('theme', 'dark-mode');
        }
    }

    // Function to apply the saved theme on page load
    (function () {
        const savedTheme = localStorage.getItem('theme') || 'light-mode'; // Default to light mode
        document.body.classList.add(savedTheme);
    })();
</script>