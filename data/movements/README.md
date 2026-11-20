# Movement database

Coach feedback, kept per movement, so that when a movement comes up again Claude can say
"last time you did this, your coach commented: ...". Comments are copied here from the Drive
videos before the videos are trashed (30 days after upload), so they outlive the videos.

## Files

- `movements.csv` — one row per movement. `id` is the key used everywhere else. `aliases` is a
  `|`-separated list of the names the coach sheet, Heath, or video titles use for it (lower case).
  `pace_sheet_id` links to `data/pace-sheet/paces.csv` where there is a pace row. Add a movement
  whenever a comment needs one that isn't here.
- `coach-comments.csv` — one row per Drive comment thread from the coach.

| column | meaning |
|---|---|
| `comment_id` | Drive comment id. Unique key: never add a second row with the same id. |
| `movement_ids` | `|`-separated ids from `movements.csv`. The movement(s) the comment is about. If the video covers several movements and the comment doesn't say which, all of them. |
| `workout_date` | Date of the session the video was recorded in (YYYY-MM-DD), not the upload date. |
| `part` | Part label from the coach sheet, e.g. `B) Back Squat 3RM`. |
| `video_title`, `video_id`, `video_url` | The Drive video. The link stops working once the video is trashed. |
| `comment_time` | When the coach posted (or last edited) the comment, UTC. |
| `author` | Comment author (the coach is Jan Kozak). |
| `comment` | The coach's words, verbatim. |
| `replies` | Later posts in the thread, `Author: text` joined with ` || `. Updated in place if new replies appear. |
| `match_method` | How the video was tied to a movement: `sheet-notes` (video link found in the JAK sheet notes row), `title` (Heath's file name), `session-notes`, `timestamp` (DJI recording time + written order), or `unmatched`. |
| `note` | Context: the load/score that day, or why the match is uncertain. |
| `recorded_at` | Date the row was added or last updated. |

## Looking up a movement

Map each movement in the day's prescription to an id (match `name`/`aliases`), then pick the rows of
`coach-comments.csv` whose `movement_ids` contain that id, newest `workout_date` first.

```sh
python3 -c "import csv,sys; [print(r['workout_date'], r['part'], '-', r['comment']) for r in sorted(csv.DictReader(open('data/movements/coach-comments.csv')), key=lambda r: r['workout_date'], reverse=True) if sys.argv[1] in r['movement_ids'].split('|')]" back-squat
```

## How comments get here

The daily scheduled task "Trash workout videos older than 30 days" harvests comments first:

1. List the videos in the Drive folder "CrossFit Workout Videos" (`1VoCQxp5ZB2h1sewAW5qn717tecVGSkIt`).
   Read comments (`read_file_content` with `includeComments: true`) on every video that was modified
   after upload (a comment bumps `modifiedTime`) and on every video about to be trashed.
2. Keep threads with at least one post not written by Heath. Skip comment ids already in the file
   (update `replies` if the thread grew).
3. Find the video's day and part: search the notes row (row 8) of the recent tabs of the JAK Google Sheet
   (`1IdokRsATshGqGFCiYw7LdhDsuCgY9OjGZ_f8olDCjYI`) for the video's file id; row 6 of that column is the date
   and the part label is the text above the link. The prescription in row 7 names the movements. Then fall back
   to the video title, the session notes in project files, and the DJI recording timestamp (videos are in the
   written order of the workout).
4. Append the rows, commit to `main`, and only then trash old videos. A video whose comments could not be saved
   is not trashed that day.
