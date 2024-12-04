<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-KyZXEAg3QhqLMpG8r+Knujsl5/0qWJ40Y+jt8brID2E6EbBMiHjE9OnPDI0zXj5e" crossorigin="anonymous">
    <title>Pagination</title>
</head>
<body>
    <div class="container" style="margin-top: 10px;">
        <?php
        try {
            $conn = new PDO("mysql:host=localhost;dbname=mysql", "root", "");
            $conn->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
        } catch (PDOException $e) {
            die("Connection failed: " . $e->getMessage());
        }

        $resultsPerPage = 5;
        $query = $conn->prepare("SELECT COUNT(*) FROM information_schema.INNODB_INDEX_STATS");
        $query->execute();
        $numberOfResults = $query->fetchColumn();
        $numberOfPages = ceil($numberOfResults / $resultsPerPage);

        $currentPage = isset($_GET['page']) ? (int)$_GET['page'] : 1;
        $currentPage = max(1, min($currentPage, $numberOfPages));

        $offset = ($currentPage - 1) * $resultsPerPage;

        $stmt = $conn->prepare("SELECT * FROM information_schema.INNODB_INDEX_STATS LIMIT :limit OFFSET :offset");
        $stmt->bindValue(':limit', $resultsPerPage, PDO::PARAM_INT);
        $stmt->bindValue(':offset', $offset, PDO::PARAM_INT);
        $stmt->execute();
        $results = $stmt->fetchAll();

        foreach ($results as $row) {
            echo $row['STAT_DESCRIPTION'] . "<br>";
        }
        ?>

        <nav aria-label="Page navigation">
            <ul class="pagination">
                <?php
                for ($page = 1; $page <= $numberOfPages; $page++) {
                    $activeClass = ($page === $currentPage) ? 'active' : '';
                    echo "<li class='page-item $activeClass'><a class='page-link' href='?page=$page'>$page</a></li>";
                }
                ?>
            </ul>
        </nav>
    </div>
</body>
</html>
