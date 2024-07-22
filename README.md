# MongoDB Task - Day 2

This repository contains MongoDB aggregation queries based on a set of predefined scenarios.

## Collections

- Users
- Codekata
- Attendance
- Topics
- Tasks
- Company Drives
- Mentors

---

## Queries

The following MongoDB queries correspond to specific tasks:

1. **Find all the topics and tasks which are thought in the month of October ?**:

   ```javascript
   db.topics.aggregate([
     {
       $lookup: {
         from: "tasks",
         localField: "_id",
         foreignField: "topic_id",
         as: "taskInfo",
       },
     },
     { $unwind: "$taskInfo" },
     {
       $match: {
         topic_taught_date: {
           $gte: new Date("2020-10-01"),
           $lt: new Date("2020-11-01"),
         },
         "taskInfo.task_taught_date": {
           $gte: new Date("2020-10-01"),
           $lt: new Date("2020-11-01"),
         },
       },
     },
     {
       $project: {
         _id: 0,
         topic_name: 1,
         topic_taught_date: {
           $dateToString: { format: "%Y-%m-%d", date: "$topic_taught_date" },
         },
         task_name: "$taskInfo.task_name",
         task_taught_date: {
           $dateToString: {
             format: "%Y-%m-%d",
             date: "$taskInfo.task_taught_date",
           },
         },
       },
     },
   ]);
   ```

2. **Find all the company drives which appeared between 15 oct-2020 and 31-oct-2020 ?**:

   ```javascript
   db.company_drives.aggregate([
     {
       $match: {
         drive_date: {
           $gte: new Date("2020-10-15"),
           $lte: new Date("2020-10-31"),
         },
       },
     },
     {
       $project: {
         _id: 0,
         drive_name: 1,
         drive_date: {
           $dateToString: { format: "%Y-%m-%d", date: "$drive_date" },
         },
       },
     },
     { $sort: { drive_date: 1 } },
   ]);
   ```

3. **Find all the company drives and students who are appeared for the placement ?**:

   ```javascript
   db.company_drives.aggregate([
     {
       $lookup: {
         from: "users",
         localField: "_id",
         foreignField: "company_drive_id",
         as: "studentInfo",
       },
     },
     { $unwind: "$studentInfo" },
     { $match: { "studentInfo.company_drive_status": "attended" } },
     {
       $project: {
         _id: 0,
         drive_name: 1,
         drive_date: {
           $dateToString: { format: "%Y-%m-%d", date: "$drive_date" },
         },
         student_name: "$studentInfo.name",
         student_email: "$studentInfo.email",
         drive_status: "$studentInfo.company_drive_status",
       },
     },
     { $sort: { drive_date: 1 } },
   ]);
   ```

4. **Find the number of problems solved by the user in codekata ?**:

   ```javascript
   db.codekata.aggregate([
     {
       $lookup: {
         from: "users",
         localField: "user_id",
         foreignField: "_id",
         as: "studentInfo",
       },
     },
     { $unwind: "$studentInfo" },
     {
       $group: {
         _id: { user_id: "$studentInfo._id", User_Name: "$studentInfo.name" },
         total_problems_solved: { $sum: 1 },
       },
     },
     {
       $project: {
         _id: 0,
         User_Name: "$_id.User_Name",
         total_problems_solved: 1,
       },
     },
     { $sort: { total_problems_solved: -1 } },
   ]);
   ```

5. **Find all the mentors with who has the mentee's count more than 15 ?**:

   ```javascript
   db.mentors.aggregate([
     { $match: { mentees_count: { $gt: 15 } } },
     {
       $project: {
         _id: 0,
         mentor_name: 1,
         mentees_count: 1,
       },
     },
     { $sort: { mentees_count: -1 } },
   ]);
   ```

6. **Find the number of users who are absent and task is not submitted between 15 oct-2020 and 31-oct-2020 ?**:

   ```javascript
   db.users.aggregate([
     {
       $lookup: {
         from: "attendance",
         localField: "_id",
         foreignField: "user_id",
         as: "attendanceInfo",
       },
     },
     {
       $lookup: {
         from: "tasks",
         localField: "_id",
         foreignField: "user_id",
         as: "taskInfo",
       },
     },
     { $unwind: "$attendanceInfo" },
     { $unwind: "$taskInfo" },
     {
       $match: {
         $and: [
           {
             "attendanceInfo.date": {
               $gte: new Date("2020-10-15"),
               $lte: new Date("2020-10-31"),
             },
           },
           { "attendanceInfo.status": "absent" },
           { "taskInfo.task_status": "not submitted" },
         ],
       },
     },
     {
       $group: {
         _id: null,
         user_count: { $sum: 1 },
       },
     },
     {
       $project: {
         _id: 0,
         task_status: "Not Submitted",
         attendance_status: "Absent",
         user_count: 1,
       },
     },
   ]);
   ```

---
