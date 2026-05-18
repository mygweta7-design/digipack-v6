DIGIPACK V9 - שליחה אוטומטית יומית

כתובת יעד לשליחה:
yossi@digipack.co.il

מה נוסף:
1. api/daily-report.js - פונקציית שרת שמייצרת דוח Excel ושולחת אותו במייל.
2. vercel.json - Cron שמפעיל את הדוח כל יום.
3. package.json - תלות ב-Supabase, Resend ו-XLSX.
4. האפליקציה נשארת PWA רגילה.

מה צריך להגדיר ב-Vercel:
Project → Settings → Environment Variables

חובה להוסיף:
SUPABASE_URL
SUPABASE_SERVICE_ROLE_KEY
RESEND_API_KEY

אופציונלי:
RESEND_FROM_EMAIL

חשוב:
לא לשים את SUPABASE_SERVICE_ROLE_KEY בתוך index.html.
הוא מיועד רק ל-Environment Variables ב-Vercel.

הערת זמן:
ב-vercel.json הוגדר 0 21 * * * שזה 21:00 UTC.
בשעון קיץ ישראל זה 00:00 בישראל.
בשעון חורף ייתכן שצריך לשנות ל-0 22 * * *.
