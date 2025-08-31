-- DROP DATABASE School_Football_Team;

CREATE DATABASE School_Football_Team;
USE School_Football_Team;

-- Player table
CREATE TABLE Players (
player_id INT PRIMARY KEY AUTO_INCREMENT,
full_name VARCHAR (50) NOT NULL,
DOB DATE NOT NULL,
positions ENUM('Goalkeeper', 'Defender', 'Midfielder', 'Striker') NOT NULL,
is_capitan BOOLEAN DEFAULT FALSE

);

-- Matches table
CREATE TABLE Matches (
match_id INT PRIMARY KEY AUTO_INCREMENT,
opponent_team_name VARCHAR (50) NOT NULL,
match_date DATE NOT NULL,
location ENUM ('Home', 'Away'),
team_score INT DEFAULT 0,
opponent_score INT DEFAULT 0
);

-- Players Statistics table
CREATE TABLE PlayerStats (
stats_id  INT PRIMARY KEY AUTO_INCREMENT,
player_id INT,
match_id INT,
goals INT DEFAULT 0 CHECK (goals >= 0),-- goals cant be negative number
assists INT DEFAULT 0  CHECK (assists >= 0), -- assists cant be negative number
fouls INT DEFAULT 0  CHECK (fouls >= 0), -- fouls cant be negative number
time_played_in_min DECIMAL (4,2) CHECK (time_played_in_min BETWEEN 0.00 and 50.00), -- school games are only 50 min long
FOREIGN KEY (player_id) REFERENCES Players(player_id),
FOREIGN KEY (match_id) REFERENCES Matches(match_id),
UNIQUE (player_id, match_id) -- no duplicate stats per player per match
);

INSERT INTO Players 
(full_name, DOB, positions, is_capitan)
VALUES
('Ivan Eroglu', '2015-04-18', 'Defender', TRUE), -- 1 
('Archie Ramage', '2014-07-22', 'Midfielder', FALSE), -- 2
('Cillian Graham', '2015-03-05', 'Striker', FALSE), -- 3
('Jamal Brown', '2015-05-13', 'Midfielder', FALSE),-- 4
('Alex Kowalski', '2014-04-03', 'Goalkeeper', FALSE),  -- 5
('Harry McLochlan', '2015-09-15', 'Midfielder', FALSE), -- 6
('Aila Stivenson', '2015-06-27', 'Midfielder', FALSE), -- 7
('William Stevenson', '2015-04-19', 'Midfielder', FALSE), -- 8
('Logan Bennett', '2014-03-20', 'Midfielder', FALSE),     -- 9 - never played
('Samuel Green', '2015-12-01', 'Striker', FALSE);         -- 10 - never played


INSERT INTO Matches
(opponent_team_name,match_date, location, team_score, opponent_score)
VALUES
('Holy Cross School', '2025-04-05', 'Home', 5, 2), -- 1
('Gailic School', '2025-04-26', 'Away', 4, 4), -- 2
('Hermitage Park', '2025-05-03', 'Away', 2, 6), -- 3
('Duddingstone Primary ', '2025-05-10', 'Home', 4, 2), -- 4
('Brunstane Primary', '2025-05-17', 'Home', 5, 1), -- 5
('Currie Primary', '2025-05-24', 'Away', 3, 3), -- 6
('Royal High School', '2025-05-31', 'Home', 3, 0), -- 7
('Blackhill Primary', '2025-06-07', 'Away', 0, 3); -- 8

-- Insert PlayerStats, some play in multiple matches
INSERT INTO PlayerStats (player_id, match_id, goals, assists, fouls, time_played_in_min) 
VALUES
(1, 1, 1, 1, 0, 48.5),  -- Ivan Eroglu (1 match)
(2, 2, 0, 2, 1, 45.0),  -- Archie Ramage (1st match)
(2, 5, 0, 1, 0, 44.0),  -- Archie Ramage (2nd match)
(3, 3, 2, 0, 2, 49.0),  -- Cillian Graham (1st match)
(3, 6, 1, 1, 0, 48.5),  -- Cillian Graham (2nd match)
(3, 8, 0, 0, 1, 46.0),  -- Cillian Graham (3rd match)
(4, 4, 1, 1, 0, 46.0),  -- Jamal Brown (1 match)
(5, 5, 0, 0, 0, 50.0),  -- Alex Kowalski (1 match)
(6, 6, 1, 0, 1, 43.5),  -- Harry McLochlan (1st match)
(6, 7, 0, 0, 0, 42.5),  -- Harry McLochlan (2nd match)
(7, 7, 0, 1, 0, 42.0),  -- Aila Stivenson (1 match)
(8, 8, 0, 0, 2, 47.0);  -- William Stevenson (1 match)

-- Show the capitan of the team
SELECT full_name, positions, DOB
FROM Players
WHERE is_capitan = TRUE;

-- Most valuable player (goals + assists together)
SELECT p.full_name, SUM(goals + assists) AS total_contribution
FROM Players p
JOIN PlayerStats ps ON p.player_id = ps.player_id
GROUP BY p.full_name
ORDER BY total_contribution DESC
LIMIT 1;

-- Total goals per player
SELECT p.full_name, SUM(ps.goals) AS total_goals
FROM Players p
JOIN PlayerStats ps ON p.player_id = ps.player_id
GROUP BY P.full_name
ORDER BY total_goals DESC;

-- Players who played in the last match. Sometimes we need to know who to pick for the next match (max 10 players per match and our team is currently 14)
SELECT p.full_name, m.match_date, m.opponent_team_name
FROM PlayerStats ps
JOIN Players p ON ps.player_id = p.player_id
JOIN Matches m ON ps.match_id = m.match_id
WHERE m.match_date = (
    SELECT MAX(match_date) FROM Matches
);

-- Top 3 players with highest average time played accross all matches
SELECT p.full_name, COUNT(ps.match_id) AS matches_played,  ROUND(AVG(ps.time_played_in_min), 2) AS avg_play_time
FROM Players p
JOIN PlayerStats ps ON p.player_id = ps.player_id
GROUP BY p.full_name
ORDER BY avg_play_time DESC
LIMIT 3;

-- Players who never played a game
SELECT full_name
FROM Players
WHERE player_id NOT IN (
    SELECT DISTINCT player_id FROM PlayerStats
);

-- Delete players who left the team
DELIMITER //

CREATE PROCEDURE DeletePlayerAndStats(IN p_id INT)
BEGIN
    DELETE FROM PlayerStats WHERE player_id = p_id;
    DELETE FROM Players WHERE player_id = p_id;
END //

DELIMITER ;

-- CALL DeletePlayerAndStats(10); -- here we just write our player_id and it will delete all the records

-- Stored Function to get match result
DELIMITER //

CREATE FUNCTION GetMatchOutcome(team_score INT, opponent_score INT)
RETURNS VARCHAR(10)
DETERMINISTIC
BEGIN
  DECLARE result VARCHAR(10);
  IF team_score > opponent_score THEN
    SET result = 'Win';
  ELSEIF team_score < opponent_score THEN
    SET result = 'Loss';
  ELSE
    SET result = 'Draw';
  END IF;
  RETURN result;
END //

DELIMITER ;

-- List of matches with final results(win/draw/loss)

SELECT match_date, opponent_team_name, team_score, opponent_score,
GetMatchOutcome(team_score, opponent_score) as Result
FROM Matches
ORDER BY match_date DESC;
