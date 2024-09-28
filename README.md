# Zen Class Program Database

This repository contains the design and implementation of a MongoDB database for the Zen class program. The database includes various collections related to students, coding tasks, attendance, topics taught, tasks assigned, company drives, and mentors.

## Database Design

### Collections

1. **`users`** - Stores user details including names, emails, coding problems solved, mentor information, attendance records, submitted tasks, and placement drives attended.
2. **`codekata`** - Contains records of coding problems solved by users.
3. **`attendance`** - Tracks user attendance with dates and status (Present/Absent).
4. **`topics`** - Lists topics taught along with the corresponding dates.
5. **`tasks`** - Records tasks assigned to users with submission dates.
6. **`company_drives`** - Contains information about placement drives, including company names and dates.
7. **`mentors`** - Stores mentor details, including the number of mentees they supervise.

## MongoDB Queries

1. **Find all topics and tasks taught in October 2020**:
   ```javascript
   db.topics.find({ taught_date: { $gte: "2020-10-01", $lte: "2020-10-31" } });
   db.tasks.find({ submission_date: { $gte: "2020-10-01", $lte: "2020-10-31" } });
   
2 Find all company drives between October 15, 2020, and October 31, 2020:
db.company_drives.find({ drive_date: { $gte: "2020-10-15", $lte: "2020-10-31" } });

3 Find all company drives and students who attended the placement:
db.company_drives.aggregate([
  {
    $lookup: {
      from: "users",
      localField: "attended_users",
      foreignField: "_id",
      as: "attended_students"
    }
  },
  {
    $project: {
      company_name: 1,
      drive_date: 1,
      attended_students: 1
    }
  }
]);

4 Find the number of problems solved by each user in CodeKata:
db.users.find({}, { name: 1, codekata_problems_solved: 1 });

5  Find all mentors with more than 15 mentees:
db.mentors.find({ mentee_count: { $gt: 15 } });

6 Count the number of users who are absent and did not submit tasks between October 15, 2020, and October 31, 2020:
db.users.aggregate([
  {
    $lookup: {
      from: "attendance",
      localField: "_id",
      foreignField: "user_id",
      as: "attendance_records"
    }
  },
  {
    $lookup: {
      from: "tasks",
      localField: "tasks_submitted",
      foreignField: "_id",
      as: "submitted_tasks"
    }
  },
  {
    $match: {
      "attendance_records.date": { $gte: "2020-10-15", $lte: "2020-10-31" },
      "attendance_records.status": "Absent",
      "submitted_tasks.submission_date": { $not: { $gte: "2020-10-15", $lte: "2020-10-31" } }
    }
  },
  {
    $count: "absent_and_task_not_submitted"
  }
]);

