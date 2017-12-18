This nifty little use-case implementation allowed me to make role changes to users matching a query using wp commmands and xargs.  In my example, I needed to get all the users that had former s2member roles (from the s2member plugin) and if any of those had made a s2member payment more than one year ago, then we remove any s2member roles and add those users to the subscriber role.

## Example

```
wp db query "select ID FROM os_users as user LEFT JOIN os_usermeta as um on um.user_id = user.ID LEFT JOIN os_usermeta as um2 on um.user_id = um2.user_id WHERE um.meta_key = 'os_s2member_last_payment_time' AND DATE_ADD(FROM_UNIXTIME(um.meta_value), INTERVAL 1 YEAR) < NOW() AND um2.meta_key = 'os_capabilities' AND um2.meta_value NOT LIKE '%s2member_level4%'" --skip-column-names --batch | xargs -n1 -I % sh -c 'wp user remove-role % s2member_level1 ; wp user remove-role % s2member_level2 ; wp user remove-role % s2member_level3 ; wp user set-role % subscriber'
```

## Breakdown

`wp db query` - used to conduct the query.  Notice I've set the `mysql` flags, `--skip-column-names` and `--batch`.  This makes sure that the results are _just_ the ids on each line (no table formatting).

`| xargs -n1 -I %` - this is used to pipe the results to `xargs` which will take each line of the result, and assign it to a % placeholder.

`sh -c` - this is basically saying I want to run a shell command and the command(s) go within quotes.  I am doing this here because I want to chain commands together for each user in the results.

`'wp user remove-role % s2member_level1 ; wp user remove-role % s2member_level2 ; wp user remove-role % s2member_level3 ; wp user set-role % subscriber'` - Here I'm simply chaining all the commands together.  Since I wasn't sure what s2member role each user would have, I simply chained through removing all roles.  I used the `;` delimiter here for each command because that indicates I want the subsequent command in the chain to run regardless of whether there was an error or not.  If I wanted the subsequent commands to _not_ run on error, I'd just use `&&` as the delimiter instead.
