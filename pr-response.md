# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

## Comment 1 — Rename
**What I did:**
Renamed `save_to_watchlist` in watchlist_service.py to `add_to_watchlist`. Then, I updated all call sites (the `add_film` function in routes/watchlist/watchlist.py).

**How I verified:**
I ran the app and tried adding a film to a specific user's watchlist with a POST request to /watchlist/<user_id>/add, and then verified the watchlist with a GET request to /watchlist/<user_id>.

## Comment 2 — Deduplication
**What I did:**
I added the following  (lines 34-41) to the `add_to_watchlist` method:
```python
# prevent duplicate watchlist entries
    existing = WatchlistEntry.query.filter_by(
        user_id=user_id, film_id=film_id
    ).first()
    if existing:
        raise AlreadyInWatchlistError(
            f"Film '{film_id}' is already in this user's watchlist"
        )
```

I modeled the logic using the equivalent lines in the `add_to_collection` method. I also added a custom exception named `AlreadyInWatchlistError`, following the setup of `add_to_collection` and its use of the `AlreadyInCollectionError`.

I then added the following error handling in the route for routes/watchlist/watchlist.py, to make use of the newly added `AlreadyInWatchlistError`. The following lines were added/modified in the routes/watchlist/watchlist.py `add_film` method:
```python
try:
        entry = add_to_watchlist(user_id=user_id, film_id=data["film_id"])
        return jsonify(entry.to_dict()), 201
    except FilmNotFoundError as e:
        return jsonify({"error": str(e)}), 404
    except AlreadyInWatchlistError as e:
        return jsonify({"error": str(e)}), 409
```

**How I verified:**
I ran the app and tried adding a film to a specific user's watchlist with a POST request to /watchlist/<user_id>/add, and did this twice to confirm that an error was returned on the second try. This indicates that the deduplication logic worked. I also used a GET request to /watchlist/<user_id> before and after each POST request to ensure the state of the user's watchlist was correct.

## Comment 3 — Missing test
**What I did:**
I created and added the content in the file `tests/test_watchlist.py`.

The test in that file is named `test_add_to_watchlist_nonexistent_film_raises`, which is modeled after the similar test `test_add_to_collection_nonexistent_film_raises` located in the `tests/test_collection.py` file.

The added test is below:
```python
def test_add_to_watchlist_nonexistent_film_raises(app, sample_user):
    """
    Adding a film_id that doesn't exist in the database should raise
    FilmNotFoundError, not a database integrity error.
    """
    with app.app_context():
        fake_film_id = "00000000-0000-0000-0000-000000000000"

        with pytest.raises(FilmNotFoundError):
            add_to_watchlist(user_id=sample_user, film_id=fake_film_id)
```

**How I verified:**
I ran `pytest tests/test_watchlist.py -v` to ensure the added test passed. 
I then ran `pytest tests/ -v` to ensure the entire test suite still passed as expected.

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
