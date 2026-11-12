# library_setup
-- =============================================================================
-- COLLEGE LIBRARY MANAGEMENT SYSTEM
-- File: library_project.sql
-- Covers: Questions C2 to C14
-- =============================================================================
create database librarydb1;
USE librarydb1;
create table members(mem_id int, mem_name varchar(20), mem_type varchar(20), joined_date date);
create table books(book_is int, author varchar(20), category varchar(20), price int, copies_available int);
create table issues(issue_id int, mem_id int, book_id int, issue_date date, due_date date, return_date date);

INSERT INTO members (mem_id, mem_name, mem_type, joined_date) VALUES
(1, 'Aarav Sharma', 'Student', '2025-01-15'),
(2, 'Priya Patel', 'Faculty', '2024-08-20'),
(3, 'Rohan Verma', 'Regular', '2025-03-10'),
(4, 'Ananya Nair', 'Student', '2025-06-05'),
(5, 'Vikram Singh', 'Premium', '2024-11-12'),
(6, 'Sneha Gupta', 'Regular', '2025-09-01'),
(7, 'Karthik Rao', 'Student', '2026-01-18'),
(8, 'Meera Joshi', 'Faculty', '2024-05-30'),
(9, 'Arjun Reddy', 'Premium', '2025-12-14'),
(10, 'Divya Menon', 'Regular', '2026-02-22');

INSERT INTO books (book_is, author, category, price, copies_available) VALUES
(101, 'R.K. Narayan', 'Fiction', 350, 5),
(102, 'Arundhati Roy', 'Fiction', 499, 3),
(103, 'A.P.J. Abdul Kalam', 'Biography', 299, 8),
(104, 'Sudha Murty', 'Literature', 250, 10),
(105, 'Chetan Bhagat', 'Romance', 220, 4),
(106, 'Amish Tripathi', 'Mythology', 399, 6),
(107, 'Vikram Seth', 'Drama', 650, 2),
(108, 'Ruskin Bond', 'Children', 199, 7),
(109, 'Jhumpa Lahiri', 'Fiction', 420, 5),
(110, 'Shashi Tharoor', 'History', 550, 3);

INSERT INTO issues (issue_id, mem_id, book_id, issue_date, due_date, return_date) VALUES
(1001, 1, 101, '2026-08-01', '2026-08-15', '2026-08-12'),
(1002, 2, 103, '2026-08-05', '2026-08-19', '2026-08-18'),
(1003, 3, 105, '2026-08-10', '2026-08-24', '2026-08-26'),
(1004, 4, 102, '2026-09-02', '2026-09-16', '2026-09-15'),
(1005, 5, 106, '2026-09-12', '2026-09-26', '2026-09-25'),
(1006, 6, 104, '2026-10-01', '2026-10-15', '2026-10-14'),
(1007, 7, 108, '2026-10-10', '2026-10-24', '2026-10-28'),
(1008, 8, 110, '2026-10-20', '2026-11-03', '2026-11-01'),
(1009, 9, 107, '2026-11-01', '2026-11-15', NULL),
(1010, 10, 109, '2026-11-05', '2026-11-19', NULL);
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
