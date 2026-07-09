# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

## Comment 1 — Rename
**What I did:** 
Renamed save_to_watchlist() to add_to_watchlist() in services/watchlist_service.py to match the codebase's existing add_to_collection() naming convention. Updated both references in routes/watchlist/watchlist.py; the import on line 8 and the call site on line 32.

**How I verified:**
 Before the rename, I searched the whole project with Get-ChildItem -Recurse -Include *.py | Select-String -Pattern "save_to_watchlist", which returned exactly three references: the definition in watchlist_service.py and the import + call site in watchlist.py. After renaming all three, I re-ran the same search and it returned zero matches, confirming nothing was missed. I then ran pytest tests/ -v and the full suite passed.

## Comment 2 — Deduplication
**What I did:**
Added deduplication to add_to_watchlist() in services/watchlist_service.py, following the existing add_to_collection() pattern in services/collection_service.py. Before creating a new WatchlistEntry, the function now queries WatchlistEntry.query.filter_by(user_id=user_id, film_id=film_id).first(). If a matching entry already exists, it raises a new AlreadyInWatchlistError instead of silently inserting a duplicate row. I defined AlreadyInWatchlistError in the watchlist service to mirror AlreadyInCollectionError in the collection service.

**Why this approach:**
I based the logic on add_to_collection(), which does the same filter_by(...).first() check and raises AlreadyInCollectionError on a hit. When a duplicate is detected, the function raises rather than no-opping, so callers get an explicit signal instead of a silent success. I noted that CollectionEntry also enforces this with a model-level UniqueConstraint, while WatchlistEntry has none. I kept the fix at the service layer as the review requested, but flagged the missing constraint as a possible future hardening. 

**How I verified:** 
Ran pytest tests/ -v; the full suite passes. The dedup path is further exercised by the watchlist tests in Comment 3 / the second stretch test.


## Comment 3 — Missing test
**What I did:**
Created tests/test_watchlist.py and added test_add_to_watchlist_nonexistent_film_raises. The test passes a film_id that doesn't exist in the database and asserts that add_to_watchlist() raises FilmNotFoundError rather than surfacing a raw database integrity error. I copied the app, sample_user, and sample_film fixtures from tests/test_collection.py so the new file follows the same isolated in-memory-DB pattern. 

**How I verified:**
Ran pytest tests/test_watchlist.py -v (the new test passes) and pytest tests/ -v (full suite passes).

**Modeled after:**
test_add_to_collection_nonexistent_film_raises in tests/test_collection.py — same fixture structure, same pytest.raises(FilmNotFoundError) assertion, adapted to the watchlist service.

## Comment 4 — Default visibility
**My position:**
**Reasoning:**
**Tradeoff acknowledged:**

## Comment 5 — Sort order
**My position:**
**Reasoning:**
**Engagement with reviewer's point:**

## Comment 6 — Rebase
**What conflicted:**
**How I resolved it:**
**How I verified no conflict remains:**

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->