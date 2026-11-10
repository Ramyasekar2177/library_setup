# library_setup
-- =============================================================================
-- COLLEGE LIBRARY MANAGEMENT SYSTEM
-- File: library_project.sql
-- Covers: Questions C2 to C14
-- =============================================================================

USE librarydb;

-- -----------------------------------------------------------------------------
-- C2: Total counts of members, books, and issue records
-- -----------------------------------------------------------------------------
SELECT 
    (SELECT COUNT(*) FROM members) AS total_members,
    (SELECT COUNT(*) FROM books) AS total_books,
    (SELECT COUNT(*) FROM issues) AS total_issues;


-- -----------------------------------------------------------------------------
-- C3: Number of titles and total copies available in each category
-- -----------------------------------------------------------------------------
SELECT 
    category,
    COUNT(book_id) AS total_titles,
    SUM(copies_available) AS total_copies_available
FROM books
GROUP BY category
ORDER BY total_copies_available DESC;


-- -----------------------------------------------------------------------------
-- C4: Top 5 most borrowed books
-- -----------------------------------------------------------------------------
SELECT 
    b.book_id,
    b.title,
    b.author,
    COUNT(i.issue_id) AS borrow_count
FROM books b
JOIN issues i ON b.book_id = i.book_id
GROUP BY b.book_id, b.title, b.author
ORDER BY borrow_count DESC
LIMIT 5;


-- -----------------------------------------------------------------------------
-- C5: Members who have never borrowed a book
-- -----------------------------------------------------------------------------
SELECT 
    m.member_id,
    m.member_name,
    m.member_type,
    m.joined_date
FROM members m
LEFT JOIN issues i ON m.member_id = i.member_id
WHERE i.issue_id IS NULL;


-- -----------------------------------------------------------------------------
-- C6: Books currently issued (not yet returned)
-- -----------------------------------------------------------------------------
SELECT 
    m.member_name,
    b.title,
    i.issue_date,
    i.due_date
FROM issues i
JOIN members m ON i.member_id = m.member_id
JOIN books b ON i.book_id = b.book_id
WHERE i.return_date IS NULL;


-- -----------------------------------------------------------------------------
-- C7: Books returned late with days late and fine calculated (Rs. 2 / day)
-- -----------------------------------------------------------------------------
SELECT 
    i.issue_id,
    i.member_id,
    i.book_id,
    i.due_date,
    i.return_date,
    DATEDIFF(i.return_date, i.due_date) AS days_late,
    (DATEDIFF(i.return_date, i.due_date) * 2) AS fine_amount
FROM issues i
WHERE i.return_date IS NOT NULL 
  AND i.return_date > i.due_date;


-- -----------------------------------------------------------------------------
-- C8: Total accumulated fine per member (only members with fines due)
-- -----------------------------------------------------------------------------
SELECT 
    m.member_id,
    m.member_name,
    SUM(DATEDIFF(i.return_date, i.due_date) * 2) AS total_fine_due
FROM members m
JOIN issues i ON m.member_id = i.member_id
WHERE i.return_date IS NOT NULL 
  AND i.return_date > i.due_date
GROUP BY m.member_id, m.member_name
HAVING total_fine_due > 0
ORDER BY total_fine_due DESC;


-- -----------------------------------------------------------------------------
-- C9: Usage comparison: Student vs Faculty
-- -----------------------------------------------------------------------------
SELECT 
    m.member_type,
    COUNT(i.issue_id) AS total_borrowed
FROM members m
JOIN issues i ON m.member_id = i.member_id
GROUP BY m.member_type;


-- -----------------------------------------------------------------------------
-- C10: Rank members by the number of books borrowed using RANK()
-- -----------------------------------------------------------------------------
SELECT 
    m.member_id,
    m.member_name,
    COUNT(i.issue_id) AS total_borrowed,
    RANK() OVER (ORDER BY COUNT(i.issue_id) DESC) AS member_rank
FROM members m
LEFT JOIN issues i ON m.member_id = i.member_id
GROUP BY m.member_id, m.member_name;


-- -----------------------------------------------------------------------------
-- C11: Most popular book category among members
-- -----------------------------------------------------------------------------
SELECT 
    b.category,
    COUNT(i.issue_id) AS total_issues
FROM books b
JOIN issues i ON b.book_id = i.book_id
GROUP BY b.category
ORDER BY total_issues DESC
LIMIT 1;


-- -----------------------------------------------------------------------------
-- C12: Books with zero copies available AND pending issues (subquery)
-- -----------------------------------------------------------------------------
SELECT 
    b.book_id,
    b.title,
    b.category,
    b.copies_available
FROM books b
WHERE b.copies_available = 0
  AND b.book_id IN (
      SELECT i.book_id
      FROM issues i
      WHERE i.return_date IS NULL
  );


-- -----------------------------------------------------------------------------
-- C13: Stored Procedure member_history(member_id)
-- -----------------------------------------------------------------------------
DROP PROCEDURE IF EXISTS member_history;

DELIMITER //

CREATE PROCEDURE member_history(IN p_member_id INT)
BEGIN
    SELECT 
        i.issue_id,
        b.title,
        b.author,
        i.issue_date,
        i.due_date,
        i.return_date,
        CASE 
            WHEN i.return_date IS NULL THEN 'Currently Borrowed'
            WHEN i.return_date > i.due_date THEN 'Returned Late'
            ELSE 'Returned On Time'
        END AS status
    FROM issues i
    JOIN books b ON i.book_id = b.book_id
    WHERE i.member_id = p_member_id
    ORDER BY i.issue_date DESC;
END //

DELIMITER ;

-- Test procedure execution:
CALL member_history(1);


-- -----------------------------------------------------------------------------
-- C14: Trigger on copy tables to reduce copies_available on new issue
-- -----------------------------------------------------------------------------
-- Recreate clean test copy tables to preserve original data
DROP TABLE IF EXISTS issues_copy;
DROP TABLE IF EXISTS books_copy;

CREATE TABLE books_copy AS SELECT * FROM books;
CREATE TABLE issues_copy AS SELECT * FROM issues;

DROP TRIGGER IF EXISTS trg_reduce_copies_available;

DELIMITER //

CREATE TRIGGER trg_reduce_copies_available
AFTER INSERT ON issues_copy
FOR EACH ROW
BEGIN
    UPDATE books_copy
    SET copies_available = copies_available - 1
    WHERE book_id = NEW.book_id;
END //

DELIMITER ;

-- Test Trigger Execution:
-- 1. Check stock before insert for book_id = 1
SELECT book_id, title, copies_available 
FROM books_copy 
WHERE book_id = 1;

-- 2. Insert a new issue record
INSERT INTO issues_copy (issue_id, member_id, book_id, issue_date, due_date, return_date)
VALUES (9999, 1, 1, CURDATE(), DATE_ADD(CURDATE(), INTERVAL 14 DAY), NULL);

-- 3. Check stock after insert (should be reduced by 1)
SELECT book_id, title, copies_available 
FROM books_copy 
WHERE book_id = 1;
