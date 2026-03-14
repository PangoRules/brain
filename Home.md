## 🔥 Active Projects
```dataview  
table start, due,
choice(due < date(today), "🔴 Overdue",
choice(due = date(today), "🟡 Due Today", "🟢 On Track")) as Status
from "Notes/Projects"
where status != "completed"
sort due asc
```


## 💡 Idea Backlog
```dataview
list from "Notes/Ideas"
where status = "raw"
sort file.ctime desc
```

## 🧠 Resources
```dataview
table author, status, started
from "Notes/Resources"
where 
    !contains(file.path, "Books/")
    OR category = "book"
sort file.name asc
```

## 📆 Today
```dataview
task
from "Notes/Projects"
where !completed
```
