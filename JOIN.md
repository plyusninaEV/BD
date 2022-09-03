# Запросы
-- Вариант 1: использование коррелирующего подзапроса
SELECT DISTINCT [b_id],
[b_name],
( [b_quantity] - (SELECT COUNT([int].[sb_book])
FROM [subscriptions] AS [int]
WHERE [int].[sb_book] = [ext].[sb_book]
AND [int].[sb_is_active] = 'Y') )
AS
[real_count]
FROM [books]
LEFT OUTER JOIN [subscriptions] AS [ext]
ON [books].[b_id] = [ext].[sb_book]
ORDER BY [real_count] DESC

-- Вариант 2: использование общего табличного выражения
-- и коррелирующего подзапроса
WITH [books_taken]
AS (SELECT [sb_book] AS [b_id],
COUNT([sb_book]) AS [taken]
FROM [subscriptions]
WHERE [sb_is_active] = 'Y'
GROUP BY [sb_book])
SELECT [b_id],
[b_name],
( [b_quantity] - ISNULL((SELECT [taken]
FROM [books_taken]
WHERE [books].[b_id] =
[books_taken].[b_id]), 0
) ) AS
[real_count]
FROM [books]
ORDER BY [real_count] DESC

-- Вариант 3: пошаговое применение общего табличного выражения и подзапроса
WITH [books_taken]
AS (SELECT [sb_book]
FROM [subscriptions]
WHERE [sb_is_active] = 'Y'),
[real_taken]
AS (SELECT [b_id],
COUNT([sb_book]) AS [taken]
FROM [books]
LEFT OUTER JOIN [books_taken]
ON [b_id] = [sb_book]
GROUP BY [b_id])
SELECT [b_id],
[b_name],
( [b_quantity] - (SELECT [taken]
FROM [real_taken]
WHERE [books].[b_id] = [real_taken].[b_id]) ) AS
[real_count]
FROM [books]
ORDER BY [real_count] DESC

-- Вариант 4: без подзапросов
WITH [books_taken]
AS (SELECT [sb_book],
COUNT([sb_book]) AS [taken]
FROM [subscriptions]
WHERE [sb_is_active] = 'Y'
GROUP BY [sb_book])
SELECT [b_id],
[b_name],
( [b_quantity] - ISNULL([taken], 0) ) AS [real_count]
FROM [books]
LEFT OUTER JOIN [books_taken]
ON [b_id] = [sb_book]
ORDER BY [real_count] DESC
