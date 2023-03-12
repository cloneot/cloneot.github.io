
# Query

## Type
- Result: `type( db.session.execute(db.select(dbname)) )`
- Row: `type( db.session.execute(db.select(dbname)).one() )`
- ScalarResult: `type( db.session.execute(db.select(dbname)).scalars() )`
- Scalar(=model object): `type( db.session.execute(db.select(dbname)).scalars().one() )`

## Result method
- `one()`: exact one or raise exception
- `fisrt()`: first or raise exception
- `all()`: all
- `one_or_none()`: exact one or none
- `scalar_one()`: `scalars().one()`
- `scalar_one_or_none()`: `scalars().one_or_none()`

