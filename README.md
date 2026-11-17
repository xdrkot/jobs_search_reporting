# Job search reporting: notes from the Gmail scan

These are the working notes from building a list of my job applications out of a Gmail mailbox,
with an AI coding assistant doing the search and extraction. They record the rules we settled on,
what the result looks like, and what went wrong on the way.

The data itself is not in this repository. See [Local files](#local-files).

## Purpose and scope

The goal was one CSV with four columns: `job title`, `company name`, `date`, `status`.

- Period scanned: January 2023 to November 2026.
- 2023 and 2024 contain no job applications. The first ones are from April 2025, and the
  activity is dense from mid-July 2025 onward.
- Final result: 322 applications. 216 are dated 2025, 97 are dated 2026, and 9 have no date
  (they come from my own list, which had no date for them).
- 5 more applications are confirmed by mail but no mail names the job title. They are kept in a
  separate "unresolved" list.

## Rules and assumptions

These were agreed step by step, mostly after a first result turned out wrong.

**What counts as an application**

- Only a submission or its confirmation counts. Feedback, status updates, interview invitations
  and rejections are not new applications; they only extend the `status` of an existing row.
- One row per position and company. The earliest evidence gives the date. A later confirmation of
  the same position becomes a note such as "re-confirmed 2026-01-27".
- A rejection with no confirmation mail still counts, because it proves an application existed.
  The status then says "No application confirmation found".
- Recruiter threads on Djinni are applications. Djinni is a platform for job seekers, so a
  recruiter writing there is answering an application or a profile I published.

**Job titles**

- Every row needs the real job title. The mail subject is not a title.
- Titles are never invented or guessed. If no mail names the position, the application goes to
  the unresolved list.

**Where the title and company are found**

| Source | Where the title is | Where the company is |
| --- | --- | --- |
| Company ATS confirmation (Workday, SuccessFactors, SmartRecruiters, Personio, Greenhouse, Ashby, join.com and others) | subject or first lines of the body | sender display name, not the sender domain |
| LinkedIn "your application was sent to X" | first line of the body after the header sentence | subject |
| LinkedIn "your application to T at X" | subject | subject |
| StepStone "deine Bewerbung als T ist raus" | subject | not stated; taken from a company mail with the same title sent within minutes |
| Djinni "Message from N, Company" | the "Job Posting" block under the message | subject |
| Agency portals such as Michael Page | body, next to a reference number | the agency; the end client is not named |
| Mail I sent myself | subject or body | recipient domain |

**Other assumptions**

- Mails are in German, English and Ukrainian, so every search needs keywords in all three.
- "Bewerbung ist raus" from StepStone is a delivery confirmation, not an outcome.
- A registration mail from a career portal, a password reset, and a "complete your application"
  reminder are not applications.
- Company names are compared after normalising: lower case, legal forms such as GmbH and AG
  removed, text in brackets removed. Titles are compared after removing gender markers such as
  (m/w/d) and reference numbers.

## Evidence sources

In the order they proved useful:

1. Automatic confirmations from company applicant-tracking systems.
2. LinkedIn "application was sent" mails.
3. StepStone confirmations.
4. Djinni recruiter threads.
5. Agency and company career portals that send their own confirmation wording.
6. The sent folder: applications I mailed directly.
7. Calendar invitations and recruiter threads, which prove an application when nothing else does.

## Conclusions

- **Mail alone is incomplete.** My hand-written list held 18 IT applications that left no trace in
  the mailbox at all, plus 5 non-IT ones. Applications through web forms without a confirmation
  mail, through messengers, or by phone cannot be recovered from mail.
- **A hand-written list alone is incomplete too.** The scan found 24 confirmed applications that I
  had not written down. Most were rejected within days, which is probably why they were never
  tracked.
- **The two sources repair each other.** My list supplied the job title for 20 applications where
  the mail named none. The mailbox supplied exact dates and outcomes for my list.
- **"Applied on" and "first mail" often differ.** Late confirmations and re-applications to the
  same position move the date by days or weeks. Where the two sources disagreed by more than two
  days, the mail date was kept and my date was noted in the status.
- **Company names do not match between sources.** The same employer appears under its legal name,
  its brand, its ATS tenant, a recruiter's name, or a typo. A small alias table was needed, and
  fuzzy matching alone was not safe.
- **A reliable report needs a manual check.** The automatic result was only good after it was
  compared against a list a person had verified.

## Experience: what went wrong and how it was fixed

| Problem | Cause | Fix |
| --- | --- | --- |
| Far too many rows | Every mail of a thread was counted | One row per position and company, earliest confirmation only |
| Titles were mail subjects | Subjects such as "Vielen Dank für Ihre Bewerbung" were copied | Title must be read from the body; otherwise the row is unresolved |
| Status mails counted as applications | Feedback and rejections looked like confirmations | Classify each mail as submission, rejection or update before counting |
| Djinni rows had no title | Only the message text was read | Read the "Job Posting" block under the message |
| StepStone mails read as rejections | "ist raus" was misunderstood | Treat as submission; match the company by title and timestamp |
| CSV columns shifted | Titles contain commas and the file was written by hand | Write with a CSV library and validate: four columns, valid date, title present, no duplicate |
| Direct applications missing | The sent folder was never searched | Add sent-mail queries; ignore replies and mail to my own addresses |
| Agency applications missing | Keyword search did not match their wording | Search by sender domain as well as by keyword |
| Latest applications missing | The scan stopped at the day of the previous run | Re-run up to today before every delivery |
| Wrong date on one row | The scan date was mistaken for the application date | Take the date from the confirmation mail only |

Smaller lessons:

- A Gmail keyword search returns far more than it should and still misses senders. Searching by
  sender domain and by exact phrase, in separate queries, works better than one large query.
- Search previews show only the oldest messages of a thread. The full thread must be opened before
  deciding the outcome.
- Rejections need care: a polite "unfortunately" in the first lines is the usual signal, but a
  confirmation can contain the same word in a disclaimer.
- An offline test with sample mails for each source catches most parsing mistakes before a real run.

## Limits

- Only one mailbox was scanned. A second address could not be connected, so applications sent
  from it are missing unless they are in my own list.
- Title and company extraction is heuristic. A sender with unusual wording can be missed or
  mislabelled.
- The end client behind an agency posting is unknown.
- 5 applications still have no title: no mail names the position and they are not in my list.
- The last part of the sent-mail results for 2026 was not read by hand.

## Local files

The following are kept locally and are deliberately not committed, because they name every
employer I applied to together with the outcome:

- the manually verified list of applications, which is the reference for everything above;
- the generated result CSV and the unresolved list;
- the Python script that scans Gmail (inbox and sent mail), applies the rules above and merges the
  manual list.
