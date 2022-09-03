# resources

A resource typically exports one thing, be it a struct or an interface with methods for interacting with the business entity.

This does not necessarily need to be "CRUD". Use names relevant to your use-case, like "JoinGroup".

Resources will also often export structs. These structs often start as a direct 1:1 mapping to a database table but over time, as more use-cases evolve, these become more and more domain-specific.
