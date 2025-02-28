# Scheduler
A system for generating a schedule of based on training, availabilty, etc.  


## Database Schema
CREATE TABLE Workers (
    WorkerID INTEGER PRIMARY KEY AUTOINCREMENT,
    FirstName TEXT NOT NULL,
    LastName TEXT NOT NULL,
	Phone TEXT,
	Email TEXT,
    Training INTEGER,  -- Comma-separated list of assignment IDs
    Availability INTEGER  -- Optional: store available times/days
);

CREATE TABLE Assignments (
    AssignmentID INTEGER PRIMARY KEY AUTOINCREMENT,
    Name TEXT NOT NULL,
    ShortName TEXT NOT NULL,
    Duration INTEGER CHECK(Duration IN (15, 30, 60, 90)),  -- Duration in minutes
    RequiredTraining INTEGER,
    Priority INTEGER NOT NULL DEFAULT 5,  -- 1 (highest priority) to 10 (lowest)
    FixedTime TEXT,  -- Stores specific start time (e.g., '14:00' for 2 PM) or NULL if flexible
    MinPerTimeSlot INTEGER DEFAULT 0,  -- Minimum required workers per block (0 = no minimum)
    MaxPerTimeSlot INTEGER DEFAULT 0  -- Maximum workers per block (0 = no limit)
);

CREATE TABLE AssignmentConstraints (
    ConstraintID INTEGER PRIMARY KEY AUTOINCREMENT,
    AssignmentA INTEGER,
    AssignmentB INTEGER,
    RestrictionType TEXT CHECK(RestrictionType IN ('Consecutive', 'SameTime', 'GapRequired')),  
    FOREIGN KEY (AssignmentA) REFERENCES Assignments(AssignmentID),
    FOREIGN KEY (AssignmentB) REFERENCES Assignments(AssignmentID)
);

CREATE TABLE ScheduleRequirements (
    RequirementID INTEGER PRIMARY KEY AUTOINCREMENT,
    Condition TEXT NOT NULL,  -- e.g., 'Wedding'
    AssignmentID INTEGER NOT NULL,
    RequiredTime TEXT,  -- e.g., '15:00' (optional, NULL means any time)
    FOREIGN KEY (AssignmentID) REFERENCES Assignments(AssignmentID)
);

CREATE TABLE Schedule (
    ScheduleID INTEGER PRIMARY KEY AUTOINCREMENT,
    Date TEXT NOT NULL,  -- YYYY-MM-DD format
    TimeSlot TEXT NOT NULL,  -- e.g., '08:00-08:30'
    WorkerID INTEGER,
    AssignmentID INTEGER,
    FOREIGN KEY (WorkerID) REFERENCES Workers(WorkerID),
    FOREIGN KEY (AssignmentID) REFERENCES Assignments(AssignmentID)
);

CREATE TABLE History (
    HistoryID INTEGER PRIMARY KEY AUTOINCREMENT,
    WorkerID INTEGER,
    AssignmentID INTEGER,
    Date TEXT NOT NULL,
    TimeSlot TEXT NOT NULL,
    FOREIGN KEY (WorkerID) REFERENCES Workers(WorkerID),
    FOREIGN KEY (AssignmentID) REFERENCES Assignments(AssignmentID)
);

CREATE TABLE TrainingTypes (
    TrainingID INTEGER PRIMARY KEY,
    Name TEXT NOT NULL,
    BitValue INTEGER UNIQUE NOT NULL  -- Powers of two: 1, 2, 4, 8, etc.
);

CREATE TABLE WorkerAvailability (
    WorkerID INTEGER,
    Date TEXT NOT NULL,  -- YYYY-MM-DD
    Available INTEGER CHECK(Available IN (0, 1)),  -- 0 = No, 1 = Yes
    PRIMARY KEY (WorkerID, Date),
    FOREIGN KEY (WorkerID) REFERENCES Workers(WorkerID)
);

