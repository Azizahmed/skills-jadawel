# Writing Arabic for Jadawel

The full glossary is `docs/GLOSSARY_AR.md` in the repository. Use its terms. If a
recurring term is missing, propose an addition to your owner instead of inventing
one.

## Conventions

- **Register.** Modern Standard Arabic and a neutral tone, with no dialect. For
  button-like labels, use the verbal noun: «إنشاء», «حفظ», «حذف».
- **Digits.** Use Western digits (0–9) everywhere: in cell values, labels and
  messages.
- **Latin tokens.** URL, API, MCP, SKU, IDs and code names stay in Latin script
  inside Arabic sentences. Leave them untranslated.
- **Placeholders.** Copy `{name}`, `{count}` and `@:action.save` exactly as they
  are.
- **Money.** For Saudi riyals, dashboards draw the riyal sign (U+20C1) to the
  left of the amount. In plain data, store numbers in number fields and leave the
  currency text out of the cell.
- **Field names.** Arabic field names are fine and expected. Pick short nouns
  («الاسم», «تاريخ الاستحقاق», «الحالة»). After creating a field, copy its name
  exactly as the schema returns it, because later writes must match it character
  for character.

## Core terms

| English | العربية |
|---|---|
| Workspace | مساحة عمل |
| Database | قاعدة بيانات |
| Table | جدول (pl. جداول) |
| Field | حقل (pl. حقول). Use عمود only for a visual grid column. |
| Row / Record | صف (in a grid) / سجل (in an expanded record) |
| View | عرض (pl. عروض) |
| Form view | نموذج |
| Page view | صفحة |
| Primary field | الحقل الأساسي |
| Dashboard / My dashboards | لوحة التحكم / لوحاتي |
| Widget | عنصر |
| Automation | أتمتة |
| Webhook | خطاف ويب |
| Template | قالب |
| Snapshot | لقطة |
| Trash | سلة المهملات |
| Member / Guest | عضو / ضيف |
| Table access | وصول الجداول |
| Viewer / Editor (table level) | مُشاهد / محرِّر |
| Protected field | حقل محمي. Avoid «مشفّر»: these fields are masked, not encrypted. |
| Mask token | رمز إخفاء |
| Endpoint protection policy | سياسة حماية الحقول |
| Billing / Plan / Subscription | الفوترة / باقة / اشتراك |
| Organization / Seat | منظمة / مقعد |
