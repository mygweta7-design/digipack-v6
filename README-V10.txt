DIGIPACK V10 FINAL

מה כלול:
1. ניהול עובדים והרשאות:
   - יצירת עובד חדש
   - שינוי סיסמה
   - השבתה / הפעלה
   - בחירת הרשאה עובד / מנהל

2. דוחות:
   - סינון לפי היום / אתמול / שבוע / חודש / שנה
   - ייצוא Excel
   - קישורי תוויות חו״ג

3. שליחה אוטומטית:
   - דוח יומי ל-yossi@digipack.co.il
   - דוח יומי ל-renana@digipack.co.il

קבצים שצריך להעלות ל-GitHub:
- index.html
- manifest.json
- service-worker.js
- icon-192.png
- icon-512.png
- package.json
- vercel.json
- התיקייה api עם הקובץ daily-report.js

משתני סביבה נדרשים ב-Vercel:
- RESEND_API_KEY
- SUPABASE_URL
- SUPABASE_SERVICE_ROLE_KEY

אופציונלי:
- RESEND_FROM_EMAIL
