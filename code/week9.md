---
title: "Week 9 Code"
layout: page
permalink: /code/week9/
---

week 9 HTML:
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{ page.title | default: site.title }}</title>
    <link rel="stylesheet" href="{{ '/assets/css/style.css' | relative_url }}">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
</head>
<body>
    <div class="container">
        <aside class="sidebar">
            <div class="about-section">

                <nav class="site-nav">
                    <a href="{{ '/' | relative_url }}" class="home-link">
                        <i class="fas fa-home"></i> Home
                    </a>
                </nav>
                
                <h2>About Me</h2>
                <img src="{{ '/assets/images/senior-picture.jpg' | relative_url }}" alt="Profile" class="profile-img">
                <p>{{ site.description }}</p>
                   </div>
    
                <!-- Social media Links/email -->
                <div class="social-links">
                    <a href="https://github.com/HenryLange/fantasy-nhl-optimizer/" target="_blank">
                        <i class="fab fa-github"></i> My GitHub repository
                    </a>
                    <a href="https://linkedin.com/in/henry-lange-253791377/" target="_blank">
                        <i class="fab fa-linkedin"></i> My LinkedIn profile
                    </a>
                    <a href="mailto:henrylange241@gmail.com">
                        <i class="fas fa-envelope"></i> My personal Email
                    </a>
                </div>
        </aside>

        <main class="main-content">
            {{ content }}
        </main>
    </div>
</body>
</html>
```
week 9 CSS:
```
---
---

@import "{{ site.theme }}";


<<<<<<< HEAD
<<<<<<< HEAD
<<<<<<< HEAD
/*overriding defaulf margins*/
=======
>>>>>>> parent of 9f528d8 (Update style.css)
=======
>>>>>>> parent of 9f528d8 (Update style.css)
=======
>>>>>>> parent of 9f528d8 (Update style.css)
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: -apple-system, BlinkMacSystemFont, sans-serif;
    line-height: 1.6;
    color: #333;
}

.container {
    display: flex;
    min-height: 100vh;
}

.sidebar {
    width: 400px;
    background-color: #f5f5f5;
    padding: 2rem;
    position: fixed;
    height: 100vh;
    overflow-y: auto;
    border-right: 1px solid #ddd;
}

.about-section h2 {
    margin-bottom: 1rem;
    color: #2c3e50;
}

.about-section {
    font-size: 1.0967em;
}

.profile-img {
    width: 200px;
    height: 200px;
    border-radius: 50%;
    object-fit: cover;
    margin-bottom: 1rem;
    display: block;
    margin-left: auto;
    margin-right: auto;
}

.social-links {
    display: flex;
    flex-direction: column;
    gap: 10px;
}

.social-links a {
    font-size: 1.67em;
}

.social-links a i {
    margin-right: 0.5rem;
    width: 20px;
    text-align: center;
}

.main-content {
    margin-left: 450px;
    margin-right: 25px;
    padding: 2rem;
    flex: 1;
}

.main-content img {
    max-width: 72%;
    max-height: 72%;
    display: block;
    margin: 20px auto;
}

pre {
    background-color: #e8eef3;
    border: 1px solid #c5d0dc;
    border-radius: 6px;
    padding: 16px;
    overflow: auto;
    font-size: 0.9em;
    line-height: 1.45;
}

code {
    background-color: #e8eef3;
    padding: 0.2em 0.4em;
    border-radius: 3px;
    font-family: monospace;
}

pre code {
    background-color: transparent;
    padding: 0;
}

.site-nav {
    margin-bottom: 20px;
    padding-bottom: 15px;
    border-bottom: 1px solid #ddd;
}

.home-link {
    font-size: 1.341em;
    text-decoration: none;
}

a {
    color: #0066cc;
}

a:visited {
    color: #0066cc;
}

a:hover {
    color: #004499;
}

.about-section h2 {
    color: #000000 !important;
}

* h1, * h3, * h4, * h5, * h6 {
    color: #000000 !important;
}

/* Responsive design */
@media (max-width: 767px) {
    .container {
        flex-direction: column;
    }
    
    .sidebar {
        position: relative;
        width: 100%;
        height: auto;
        border-right: none;
        border-bottom: 1px solid #ddd;
    }
    
    .main-content {
        margin-left: 0;
    }
}
```
### [return to week 9 post](https://henrylange.github.io/fantasy-nhl-optimizer/2025/11/02/week-nine.html)
