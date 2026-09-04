# blaze-xpert-academy

**Domain:** Academy/Education. **Type:** Odoo customization project (bespoke, not a shared
product).
**Repo:** `xperts_academy` (`git@bitbucket.org:teapottechies/xperts_academy.git`, branch `17.0`)
— in the `odoo-17/tpt-ind` workspace.
**Host:** `ap-southeast-1` (standard backend, threaded). Has a real domain + nginx vhost.

## What's known from code

8 modules: `hr_employee_extend` (the core "Xpert Academy" app for student/batch/fee management,
built on `hr` + `sale_subscription`), `xpert_registration` (public registration flow),
`xperts_web` (the client's public website), plus India-localization/invoicing tweaks
(`account_l10n_in_inherit`, `l10n_in_report_hide_empty_hsn`, `account_report_hide_empty_taxes`,
`xa_global_payment_term`) and `data_migration` (a one-off migration module, not permanent
architecture).

**Note:** `hr_employee_extend` and `xpert_registration` depend on `sale_subscription`, an
**Odoo Enterprise** module not vendored in this checkout — installing either requires Enterprise
on the `addons_path`.

## Engagement / business context

Not yet documented — TODO: contacts, engagement history, Enterprise licensing arrangement.
