<div dir="rtl">

# LoVeT ClinicAI — מערכת ניהול טיקטים

מערכת הטיקטים המרכזית לפרויקט LoVeT — מערכת ניהול מרפאה וטרינרית.

**[פתח את לוח הקנבאן](https://github.com/orgs/codama-dev/projects/2/views/2)**

## איך פותחים טיקט?

1. לחצו על **[טיקט חדש](../../issues/new/choose)**
2. בחרו תבנית:
   - **Bug Report** — משהו לא עובד (באג)
   - **Feature Request** — בקשה לפיצ'ר חדש
   - **Improvement** — שיפור לפיצ'ר קיים
3. מלאו את הטופס ושלחו

הטיקט יופיע אוטומטית בלוח הקנבאן בעמודת **Backlog**.

## תהליך העבודה

```
Backlog → Ready for Work → In Progress → Ready for Product Test → Done
```

| עמודה | מה קורה כאן |
|---|---|
| **Backlog** | טיקט חדש, טרם תועדף |
| **Ready for Work** | תועדף ומוכן לפיתוח |
| **In Progress** | בפיתוח פעיל |
| **Ready for Product Test** | הפיתוח הסתיים, מחכה לבדיקת מוצר |
| **Done** | אושר ונסגר |

## מה עושים אחרי בדיקת מוצר?

- **עבר בהצלחה** — גררו את הכרטיס ל-**Done**
- **לא עבר** — גררו חזרה ל-**Ready for Work**. המערכת סופרת אוטומטית כמה פעמים טיקט חזר (שדה Rejections) ומוסיפה תגובה בטיקט.

## טיפים לדיווח באגים טוב

- **צילום מסך** — תמונה שווה אלף מילים! גררו תמונות ישירות לטופס
- **לוגים מהקונסול** — לחצו F12 (או Cmd+Option+J במק), העתיקו שגיאות אדומות והדביקו
- **שגיאות רשת** — בלשונית Network בקונסול, חפשו בקשות אדומות

## שדות בכרטיס

| שדה | אפשרויות |
|---|---|
| **Priority** | Critical / High / Medium / Low |
| **Area** | Frontend / Backend / Full Stack |
| **Category** | Feature / Bug / Refactor / Chore |
| **Rejections** | מספר — עולה אוטומטית כשטיקט חוזר מבדיקה |

## ריפוזטוריז קשורים

| ריפו | תיאור |
|---|---|
| [clinic-ai-fe](https://github.com/codama-dev/clinic-ai-fe) | פרונטאנד — React + Vite |
| [lovet-express-backend](https://github.com/codama-dev/lovet-express-backend) | באקאנד — Express + Prisma |

</div>
