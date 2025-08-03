
 - Open world: do we want to design this in a way that "Deny" is not a part of the auth system to
   ensure that when integrating with external data don't accidentally overpermission. I suspect rather than
   doing this we will be in a position where it is consistently deciding Y/N for entire graphs to be merged
   into the graph for permission evaluation
 - Support query creation: When an entity goes to access a resource - if the ACL graph does not currently satisfy
   the generation of queries that, e.g., can be issued using the DCQL
 - Similar to SHACL, we may wish to have a "permissions" graph and a "properties" graph wherin the permissions
   graph (or at least distinct entites that are allowed to make "permissions" statements and "properties" statemements)
   such that you can't just come in with a statement from the gov't and say "I have permission to access this resource".
 - Actually I *think* the model should be that you start at top level saying who is permitted to

 - We may want to support an "access hint" API which enables clients to request 
