# Mongoose Schema Migrations

Performs mongoose data migrations on document init.

## Brief Example

```javascript
var schema = new mongoose.Schema({ /* ... */ });

var currentSchemaVersion = 2;
var opts = {
    versionKey: '__sv',   // The default
    saveOnMigration: true // Also the default
};

migrations(schema, currentSchemaVersion, opts).add({
    version: 1,
    up: function() {
        var data = this;
        var nameParts = data.name.split(' ', 2);
        data.firstName = nameParts[0];
        data.lastName = nameParts[1];
        delete data.name;
    },
    down: function() {
        var data = this;
        data.name = data.firstName + ' ' + data.lastName;
        delete data.firstName;
        delete data.lastName;
    }
}).add({
    version: 2,
    up: function() {
        var data = this;
        var lastNameParts = data.lastName.split(',', 2);
        data.suffix = null;
        if (2 == lastNameParts.length) {
            data.lastName = lastNameParts[0];
            data.suffix = lastNameParts[1].replace(/^\s/, '');
        }
    },
    down: function() {
        var data = this;
        if (data.suffix) {
            data.lastName += ', ' + data.suffix;
            delete data.suffix;
        }
    }
});
```


## Important Notes

- Mongoose (4) does not load fields which are not in the schema.  So in the above example, if the `name` key had been removed from the schema, the migration would be impossible, and the functions would throw an error on `data.name.split()`.

  Setting strict mode to false and running migrations in a pre-init hook still did not give us that missing field.

- The `delete data.field;` method above might not work for you.
  If the field exists in the schema, then `data.field = undefined;` might work.  (It did for me under Mongoose 4.)
  Otherwise, to be sure the field gets deleted, use: `data.set('field', undefined, { strict: false });` instead.
  See the discussion in the comments of [this SO answer](http://stackoverflow.com/a/6938733/99777).


## Other Thoughts

There is no suppoert for async or multi-collection migrations (imagine you wanted to split one collection into two, or merge two collections into one).  But supporting async single-document migrations could be troublesome anyway: trying to migrate two models which keep loading each other would create an infinite loop!
