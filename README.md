<?php
// Last.fm API key
$apiKey = '296176cf7d56fa78648ff38a26a22e11';

// Fetching the top tracks from Last.fm
$topTracksUrl = "https://ws.audioscrobbler.com/2.0/?method=chart.gettoptracks&api_key=$apiKey&format=json";

$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, $topTracksUrl);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);

$response = curl_exec($ch);
curl_close($ch);

$topTracksData = json_decode($response, true);

// NewsAPI key
$newsApiKey = 'a0478c619dc74354a897ca3fb0a502a4';
$domains = 'billboard.com, rollingstone.com';

$newsUrl = "https://newsapi.org/v2/everything?" . http_build_query([
    'q' => 'Music',
    'language' => 'en',
    'sortBy' => 'publishedAt',
    'pageSize' => 20,
    'domains' => $domains,
    'apiKey' => $newsApiKey,
]);

// Initialize cURL for NewsAPI
$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, $newsUrl);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HTTPHEADER, [
    'User-Agent: MusicNewsApp/1.0',
]);

$response = curl_exec($ch);
curl_close($ch);

if (!$response) {
    echo "<p>Failed to connect to the NewsAPI service. Please check your network connection.</p>";
    exit;
}

$newsData = json_decode($response, true);

if ($newsData['status'] !== 'ok' || empty($newsData['articles'])) {
    echo "<p>Error from NewsAPI: " . htmlspecialchars($newsData['message'] ?? 'No articles found.') . "</p>";
    exit;
}
?>

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Music News</title>
    <link href="https://maxcdn.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css" rel="stylesheet">
    <style>
        html, body {
            height: 100%;
            margin: 0;
            display: flex;
            flex-direction: column;
            background-color: black;
        }
        .content {
            flex: 1;
        }
        .trending-music {
            margin-top: 40px;
            padding: 20px;
            background-color: #f8f9fa;
            border-radius: 5px; /* Added for better aesthetics */
        }
        .trending-songs-list {
            max-height: 450px; /* Set the maximum height */
            overflow-y: auto;   /* Enable vertical scrolling */
            margin-top: 20px;   /* Margin above the list */
        }
        footer {
            margin-top: auto;
        }
        header {
            background-color: #6f42c1; /* Purple header */
        }
        nav a {
        text-decoration: none;
        color: white; /* Initial text color */
        transition: color 0.3s ease; /* Smooth transition for color change */
        }
        nav a:hover {
            color: green; /* Spotify green color on hover */
            text-decoration: none;
        }   
    </style>
</head>
<body>

<header class="text-white text-center py-3">
    <nav>
    <a href="MusicPage.php" class="mx-2">Home</a>
    <a href="#" class="mx-2">About</a>
    <a href="https://open.spotify.com/" class="mx-2">Spotify</a>
</nav>
</header>
<br>
<h3 style="text-align: center; 
            color: black; 
            text-shadow: 0 0 1px green, 0 0 2px purple; 
            animation: glow 3s infinite alternate;">
    SURF THROUGH THE MUSIC INDUSTRY
</h3>

<style>
@keyframes glow {
    0% {
        text-shadow: 0 0 1px cyan, 0 0 2px cyan;
        color: black;
    }
    50% {
        text-shadow: 0 0 1px green, 0 0 2px green;
        color: green;
    }
    100% {
        text-shadow: 0 0 1px purple, 0 0 2px purple;
        color: purple;
    }
}
</style>

<br>
<div id="newsCarousel" class="carousel slide" data-ride="carousel" data-interval="3000">
    <div class="carousel-inner">
        <?php foreach ($newsData['articles'] as $index => $article): ?>
            <div class="carousel-item <?= $index === 0 ? 'active' : '' ?>">
                <div class="row">
                    <div class="col-md-6">
                        <img src="<?= htmlspecialchars($article['urlToImage'] ?? 'default-image.jpg') ?>" class="d-block w-100" alt="News Image" style="max-height: 400px; object-fit: cover;">
                    </div>
                    <div class="col-md-6" style="color: white;"> <!-- Make text white -->
                        <h4 style="color: white;"><?= htmlspecialchars($article['title']) ?></h4>
                        <p><strong style="color: white;"><?= htmlspecialchars($article['source']['name']) ?></strong> - <?= date('F j, Y', strtotime($article['publishedAt'])) ?></p>
                        <p style="color: white;"><?= htmlspecialchars($article['description']) ?></p>
                        <a href="<?= htmlspecialchars($article['url']) ?>" target="_blank" class="btn" style="background-color: green; color: white;">Read more</a>
                    </div>
                </div>
            </div>
        <?php endforeach; ?>
    </div>
</div>
<div class="container mt-5 content">
        </div>

        <!-- Carousel controls -->
        <a class="carousel-control-prev" href="#newsCarousel" role="button" data-slide="prev">
            <span class="carousel-control-prev-icon" aria-hidden="true"></span>
            <span class="sr-only">Previous</span>
        </a>
        <a class="carousel-control-next" href="#newsCarousel" role="button" data-slide="next">
            <span class="carousel-control-next-icon" aria-hidden="true"></span>
            <span class="sr-only">Next</span>
        </a>
    </div>

    <!-- Trending music section -->
    <div class="trending-music mt-1">
        <h3 class="text-center">Top Trending Songs</h3>
        <div class="trending-songs-list">
            <ul class="list-group">
                <?php if (!empty($topTracksData['tracks']['track'])): ?>
                    <?php foreach ($topTracksData['tracks']['track'] as $index => $track): ?>
                        <?php 
                        $songName = htmlspecialchars($track['name']);
                        $artistName = htmlspecialchars($track['artist']['name']);
                        // Create a YouTube search URL
                        $youtubeSearchUrl = "https://www.youtube.com/results?search_query=" . urlencode($songName . " " . $artistName);
                        ?>
                        <li class="list-group-item">
                            <a href="<?= $youtubeSearchUrl ?>" target="_blank"><?= $songName ?> - <?= $artistName ?></a>
                            <span class="badge badge-primary float-right"><?= $index + 1 ?></span>
                        </li>
                    <?php endforeach; ?>
                <?php else: ?>
                    <li class="list-group-item">No trending songs available.</li>
                <?php endif; ?>
            </ul>
        </div>
    </div>
</div>
<footer style="background-color: #6f42c1; color: white; text-align: center; padding: 1rem; margin-top: 2rem;">
    <p>&copy; <?= date('Y') ?> Music News Portal. All rights reserved.</p>
</footer>

<!-- Bootstrap JS and dependencies -->
<script src="https://code.jquery.com/jquery-3.5.1.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@popperjs/core@2.5.3/dist/umd/popper.min.js"></script>
<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/js/bootstrap.min.js"></script>

</body>
</html>
