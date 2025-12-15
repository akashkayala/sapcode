1. Fetch user details

Selects user name (bname) and valid-to date (gltgb) from table USR02

Only dialog users (USTYP = 'A') are considered

User selection is restricted using a selection range (s_user)



2. Check data availability

Validates whether the SELECT statement fetched records using sy-subrc

Processing continues only if data is available



3. Loop through user records

Loops over each entry in internal table gt_users

Clears the final work area before populating new values



4. Validate user validity date

Checks if the user’s validity end date (gltgb) is greater than or equal to system date

Ensures only currently active users are processed



5. Generate email ID for valid users

Concatenates the user name with company email domain (@cadence.com)

Converts the email ID to lower case

Stores the generated email ID in final structure



6. Handle expired users

If the validity date is less than system date

Email ID is still generated using the same domain

Converts email ID to lower case for consistency



7. Populate final internal table

Appends the processed user data into gt_final

Clears the work area after each append to avoid data overlap



8. End of processing

Completes looping and exits FORM after data preparation
