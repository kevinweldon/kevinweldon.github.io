<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Kevin Weldon</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: Georgia, 'Times New Roman', serif;
            line-height: 1.6;
            color: #333;
            background-color: #fff;
            padding: 40px 20px;
            max-width: 800px;
            margin: 0 auto;
        }

        header {
            margin-bottom: 60px;
            text-align: center;
        }

        h1 {
            font-size: 2.5rem;
            font-weight: normal;
            margin-bottom: 10px;
            letter-spacing: -0.02em;
        }

        .subtitle {
            font-size: 1.1rem;
            color: #666;
            font-style: italic;
        }

        nav {
            margin: 40px 0 60px 0;
            text-align: center;
        }

        nav a {
            color: #333;
            text-decoration: none;
            margin: 0 20px;
            font-size: 1.1rem;
            border-bottom: 1px solid transparent;
            transition: border-bottom 0.2s ease;
        }

        nav a:hover {
            border-bottom: 1px solid #333;
        }

        .intro {
            font-size: 1.2rem;
            margin-bottom: 40px;
            line-height: 1.7;
        }

        .section {
            margin-bottom: 50px;
        }

        .section h2 {
            font-size: 1.5rem;
            margin-bottom: 20px;
            font-weight: normal;
            border-bottom: 1px solid #eee;
            padding-bottom: 10px;
        }

        .project-list, .post-list {
            list-style: none;
        }

        .project-list li, .post-list li {
            margin-bottom: 15px;
            padding: 15px 0;
            border-bottom: 1px solid #f5f5f5;
        }

        .project-list li:last-child, .post-list li:last-child {
            border-bottom: none;
        }

        .project-title, .post-title {
            font-size: 1.1rem;
            font-weight: bold;
            margin-bottom: 5px;
        }

        .project-description, .post-excerpt {
            color: #666;
            font-size: 0.95rem;
        }

        .date {
            color: #999;
            font-size: 0.9rem;
            font-style: italic;
        }

        a {
            color: #333;
            text-decoration: underline;
            text-decoration-color: #ccc;
        }

        a:hover {
            text-decoration-color: #333;
        }

        .photo-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
            gap: 20px;
            margin-top: 20px;
        }

        .photo-placeholder {
            background-color: #f5f5f5;
            height: 200px;
            display: flex;
            align-items: center;
            justify-content: center;
            color: #999;
            font-style: italic;
        }

        footer {
            margin-top: 80px;
            padding-top: 20px;
            border-top: 1px solid #eee;
            text-align: center;
            color: #666;
            font-size: 0.9rem;
        }

        @media (max-width: 600px) {
            body {
                padding: 20px 15px;
            }
            
            h1 {
                font-size: 2rem;
            }
            
            nav a {
                display: block;
                margin: 10px 0;
            }
            
            .intro {
                font-size: 1.1rem;
            }
        }
    </style>
</head>
<body>
    <header>
        <h1>Kevin W.</h1>
        <p class="subtitle">welcome to my site</p>
    </header>

    <nav>
        <a href="#projects">Projects</a>
        <a href="#writing">Writing</a>
        <a href="https://kevinweldon.github.io/photography">Photography</a>
        <a href="#about">About</a>
    </nav>

    <main>
        <h2>Photography</h2>
        <div class="photo-grid">
            <div class="photo-placeholder">Photo 1</div>
            <div class="photo-placeholder">Photo 2</div>
            <div class="photo-placeholder">Photo 3</div>
            <div class="photo-placeholder">Photo 4</div>
        </div>

    </main>

    <footer>
        <p>coded with html on github</p>
    </footer>
</body>
</html>