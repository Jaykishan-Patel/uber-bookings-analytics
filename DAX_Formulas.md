This document contains all the **DAX measures and calculations** used in the **Uber Trip Analysis Power BI Dashboard**, grouped by analysis category.  

---

## 📅 Date & Calendar Measures  

### **Calendar Table**  
```DAX
Calender_Table = 
CALENDAR(MIN('Trip Details'[PickUp_Date]), MAX('Trip Details'[PickUp_Date]))
