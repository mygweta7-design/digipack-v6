import { createClient } from "@supabase/supabase-js";
import { Resend } from "resend";
import * as XLSX from "xlsx";

const RECIPIENT_EMAILS = ["yossi@digipack.co.il", "renana@digipack.co.il"];
const TIMEZONE = "Asia/Jerusalem";

function getIsraelDayRangeForYesterday() {
  // Cron is scheduled for 21:00 UTC, which is usually 00:00 Israel time during DST.
  // To make the daily report useful, this function sends the report for the previous Israel calendar day.
  const now = new Date();
  const israelNow = new Date(now.toLocaleString("en-US", { timeZone: TIMEZONE }));
  israelNow.setDate(israelNow.getDate() - 1);
  israelNow.setHours(0, 0, 0, 0);

  const israelEnd = new Date(israelNow);
  israelEnd.setHours(23, 59, 59, 999);

  const startIso = new Date(israelNow.getTime() - israelNow.getTimezoneOffset() * 60000).toISOString();
  const endIso = new Date(israelEnd.getTime() - israelEnd.getTimezoneOffset() * 60000).toISOString();

  // Fallback date label
  const label = israelNow.toLocaleDateString("he-IL");
  return { startIso, endIso, label };
}

function formatDate(value) {
  if (!value) return "";
  return new Date(value).toLocaleDateString("he-IL", { timeZone: TIMEZONE });
}

function formatTime(value) {
  if (!value) return "";
  return new Date(value).toLocaleTimeString("he-IL", { timeZone: TIMEZONE });
}

export default async function handler(req, res) {
  try {
    if (!process.env.SUPABASE_URL || !process.env.SUPABASE_SERVICE_ROLE_KEY || !process.env.RESEND_API_KEY) {
      return res.status(500).json({
        ok: false,
        error: "Missing environment variables. Required: SUPABASE_URL, SUPABASE_SERVICE_ROLE_KEY, RESEND_API_KEY"
      });
    }

    const supabase = createClient(
      process.env.SUPABASE_URL,
      process.env.SUPABASE_SERVICE_ROLE_KEY
    );

    const resend = new Resend(process.env.RESEND_API_KEY);
    const { startIso, endIso, label } = getIsraelDayRangeForYesterday();

    const { data: jobs, error } = await supabase
      .from("jobs")
      .select("*, employees(name)")
      .gte("created_at", startIso)
      .lte("created_at", endIso)
      .order("created_at", { ascending: false });

    if (error) throw error;

    const jobIds = (jobs || []).map(j => j.id).filter(Boolean);
    let labelsByJob = {};

    if (jobIds.length) {
      const { data: labels, error: labelsError } = await supabase
        .from("material_labels")
        .select("*")
        .in("job_id", jobIds)
        .order("created_at", { ascending: true });

      if (labelsError) throw labelsError;

      (labels || []).forEach(labelRow => {
        if (!labelsByJob[labelRow.job_id]) labelsByJob[labelRow.job_id] = [];
        labelsByJob[labelRow.job_id].push(labelRow);
      });
    }

    const rows = (jobs || []).map(job => {
      const labels = labelsByJob[job.id] || [];
      const firstStartLabel = labels.find(l => l.label_type === "start" && l.image_url)?.image_url || "";
      const allLabelLinks = labels.filter(l => l.image_url).map(l => l.image_url).join(" | ");

      return {
        "תאריך": formatDate(job.created_at),
        "שעת התחלה": formatTime(job.start_time),
        "שעת סיום": formatTime(job.end_time),
        "שם מפעיל": job.employees?.name || "",
        "תהליך": job.department || "למינציה",
        "פק״ע": job.batch_number || job.order_number || "",
        "כמות מטרים להעברה": job.transfer_meters || "",
        "סטטוס": job.status || "",
        "תווית חו״ג ראשית": firstStartLabel,
        "כל תוויות חו״ג": allLabelLinks
      };
    });

    const worksheet = XLSX.utils.json_to_sheet(rows);
    const workbook = XLSX.utils.book_new();
    XLSX.utils.book_append_sheet(workbook, worksheet, "דוח יומי");
    const excelBuffer = XLSX.write(workbook, { bookType: "xlsx", type: "buffer" });

    const subject = `DIGIPACK - דוח יומי ${label}`;

    await resend.emails.send({
      from: process.env.RESEND_FROM_EMAIL || "DIGIPACK <onboarding@resend.dev>",
      to: RECIPIENT_EMAILS,
      subject,
      html: `
        <div dir="rtl" style="font-family:Arial,sans-serif">
          <h2>DIGIPACK - דוח יומי</h2>
          <p>תאריך דוח: ${label}</p>
          <p>מספר רשומות: ${rows.length}</p>
          <p>מצורף קובץ Excel.</p>
        </div>
      `,
      attachments: [
        {
          filename: `digipack-daily-report-${label.replaceAll("/", "-")}.xlsx`,
          content: excelBuffer.toString("base64")
        }
      ]
    });

    return res.status(200).json({
      ok: true,
      sentTo: RECIPIENT_EMAILS,
      rows: rows.length,
      date: label
    });
  } catch (err) {
    return res.status(500).json({
      ok: false,
      error: err.message || String(err)
    });
  }
}
