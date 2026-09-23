## Сутності та атрибути

### Parent 
- `parent_id`
- `first_name`
- `second_name`
- `email`
- `phone_number`

### Student
- `student_id`
- `first_name`
- `second_name`
- `birth_date`
- `grade_level`
- `email`
- `phone_number`
- `parent_id`

### Teacher
- `teacher_id`
- `first_name`
- `second_name`
- `email`
- `specialization`

### Course
- `course_id`
- `title`
- `specialization`
- `level`
- `description`
- `schedule`
- `price`
- `teacher_id`

### Lesson
- `lesson_id`
- `course_id`
- `schedule`
- `link`

### Enrollment
- `enrollment_id`
- `student_id`
- `course_id`
- `enrolled_at`
- `status`
- `price`

### Homework
- `homework_id`
- `lesson_id`
- `title`
- `description`
- `due_date`

### Homework_submission
- `homework_submission_id`
- `student_id`
- `homework_id`
- `submitted_at`
- `grade`
- `feedback`
- `status`

### Subscription
- `subscription_id`
- `student_id`
- `course_id`
- `start_date`
- `end_date`
- `payment_method`
- `price`

## Зв'язки

|      Звʼязок     | Кардинальність |  Пояснення  |
| ---------------- | -------------- | ----------- |
| Parent - Student | 1 - N      | Батьки можуть мати кількох дітей, одна дитина має одного опікуна |
