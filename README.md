README: 

Question 1.

When designing this schema, I prioritized the ease of answering key questions, such as finding which rooms are available on a given day for Hotel A or which finding the number of check-ins on a given day, with only a few joins. 

I created separate tables for reservations and bookings (A guest can book several rooms within the same reservation but only one room per booking) and connected them with the reservation ID as the foreign key. Additionally, since rooms can be booked multiple times, there is a many-to-one relationship between the booking and room tables. 

We can create an index on room numbers, and depending on how many hotels there are, index the cities the hotel are located for faster queries. 

Instead of a room type table, I would establish a hotel grading/tiering system that is standardized across hotels as well as a room tiering system that is standardized within each hotel, which is more adaptable and scalable. Because this is a global chain, different brands within the chain will likely have different tiers depending on the hotel size, purpose, and demographic.  For example, a hotel like the Ritz-Carlton would be Tier 5 whilst a motel would be Tier 1, and a Tier 5 room would be a master suite while a Tier 1 room would be a basic room. By joining the hotel and and room tables on Hotel ID, you can look at the hotel tier and room tier side-by-side and instantly have a price estimate of a booking.

Other Notes:

Reservation status (enum) indicates a confirmation, cancellation, no-show, etc. Cancellation deadline is dependent on payment date listed in the payment table.

Question 2.

I used PostgreSQL to run queries + pgAdmin 4 as my server. 

I cleaned up duplicate emails from ‘persons_data’ table and create a new table ‘persons’ and did the same with names in ‘majors_data’, which I stored in ‘majors’, keeping the first occurrences for both tables. To preserve the order of the rows, I added a rownumber column to the original tables and filtered the first occurences with the following query: 

Example query:
```
CREATE TABLE persons AS
WITH CTE AS(
	SELECT *,
	ROW_NUMBER() OVER(PARTITION BY email
	ORDER BY rownum) AS RowNumber
	FROM persons_data
)
SELECT * FROM CTE WHERE RowNumber = 1;
```
I included the csv files of the unused data (duplicate occurrences for persons_data and major_data in the files persons_duplicate and majors_duplicate respectively). 

My query:
```
CREATE TABLE housing_table AS
SELECT p.personId, CONCAT(firstname, ' ', lastname) as name, 
	email, dob,
	SPLIT_PART(address, ', ', 1) AS address1,
	SPLIT_PART(address, ', ', 2) AS city,
	SPLIT_PART(address, ', ', 3) AS state,
	STRING_AGG(m.id, ', ') AS majorIds,
	i.bedId
FROM persons p
LEFT JOIN LATERAL STRING_TO_TABLE(majors, ', ') as mjr ON true
LEFT JOIN majors_table m ON mjr = m.name
LEFT JOIN occupancy_data o ON p.personId = o.personId
LEFT JOIN inventory_data i ON o.roomName = i.roomName AND o.bedName = i.bedName
GROUP BY p.personId, firstname, lastname, email, dob, address, i.bedId;
```

I did not include ‘zip’ column in final table. USPS has ZipCodeLookup API (https://www.usps.com/business/web-tools-apis/address-information-api.html) or API from SmartyStreets (https://www.smarty.com/docs/cloud/us-zipcode-api) or ZipCodeAPI (https://www.zipcodeapi.com/API#locToZips).
I could query the API w/Python, which returns responses as JSON data, format the data with pandas, and either update the table with psycopg2 (PostgreSQL adapter) or manually import the column in my server. 
