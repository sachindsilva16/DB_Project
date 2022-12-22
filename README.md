create database matrimony_database1;
use matrimony_database1;

create table USER_ACCOUNT(Userid int primary key,
Fname varchar(20), 
Lname varchar(20), 
Age int,
Height_in_cm int,
Job varchar(20),
DOB date,
Gender char(5),
Mother_Tongue varchar(20) ,
Num_of_matches int)

insert into USER_ACCOUNT values(201,'Sima','Hegde', 27, 152, 'Musician', '1995-03-24', 'F','Tulu',NULL)
insert into USER_ACCOUNT values(202,'Deepak','Bhatt', 34, 160, 'Architect', '1988-12-25', 'M','Kannada',NULL)
insert into USER_ACCOUNT values(203,'Zoya','Ghose', 26, 165, 'Lawyer', '1996-01-11', 'F','Kannada',NULL)
insert into USER_ACCOUNT values(204,'Vritika','Dyal', 29, 169, 'Artist', '1992-12-08', 'F','Kannada',NULL)
insert into USER_ACCOUNT values(205,'Azad','Kumar', 30, 188, 'Photographer', '1993-06-06', 'M','Kannada',NULL)
insert into USER_ACCOUNT values(206,'Kalyani','More', 28, 155, 'Youtuber', '1994-05-29', 'F','Kannada',NULL)
insert into USER_ACCOUNT values(207,'Murali','Andra', 34, 190, 'Freelancer', '1988-04-30', 'M','Hindi',NULL)
insert into USER_ACCOUNT values(208,'Kapil','Dev', 35, 187, 'Actress', '1989-04-30', 'M','Hindi',NULL)
insert into USER_ACCOUNT values(209,'Murali','Agarwal', 24, 158, 'Dentist', '1998-04-30', 'M','Hindi',NULL)
insert into USER_ACCOUNT values(210,'Oliver','Andra', 21, 178, 'Police', '1978-03-24', 'M','Kannada',NULL)
insert into USER_ACCOUNT values(211,'Mary','Ahuja', 27, 177, 'Dentist', '1984-06-15', 'F','Hindi',NULL)
insert into USER_ACCOUNT values(212,'Marge','Agarwal', 26, 189, 'Teacher', '1983-07-16', 'F','Hindi',NULL)
insert into USER_ACCOUNT values(213,'Lisa','Ahuja', 25, 187, 'Pilot', '1993-08-13', 'F','Tulu',NULL)
insert into USER_ACCOUNT values(214,'Katy','Reddy', 32, 166, 'Baker', '1981-03-15', 'F','Hindi',NULL)
insert into USER_ACCOUNT values(215,'Thomas','Patel', 22, 183, 'Designer', '1986-05-25', 'M','Tulu',NULL)
insert into USER_ACCOUNT values(216,'Jaswin','Anand', 28, 190, 'Freelancer', '1988-04-23', 'M','Hindi',NULL)

select * from USER_ACCOUNT;


create table PREFERENCES(Userid int primary key,
Mother_Tongue varchar(20), Gender char(5),
Min_Age int, Max_Age int, Min_Height int,
Max_Height int ,
constraint fk1 foreign key (Userid) references USER_ACCOUNT(Userid))



insert into PREFERENCES values(201,'Tulu', 'M',21,31,174,184)
insert into PREFERENCES values(202,'Kannada', 'F',22,27,156,178)
insert into PREFERENCES values(203,'Kannada', 'M',21,29,178,190)
insert into PREFERENCES values(204,'Kannada', 'M',23,31,157,190)
insert into PREFERENCES values(205,'Kannada', 'F',22,35,145,179)
insert into PREFERENCES values(206,'Kannada', 'M',22,35,165,174)
insert into PREFERENCES values(207,NULL, 'F',29,35,174,179)
insert into PREFERENCES values(208,'Hindi', 'F',24,30,181,190)
insert into PREFERENCES values(209,'Hindi', 'F',26,28,164,190)
insert into PREFERENCES values(210,'Kannada', 'F',24,28,158,176)
insert into PREFERENCES values(211,'Hindi', 'M',24,29,175,185)
insert into PREFERENCES values(212,'Hindi', 'M',28,34,145,155)
insert into PREFERENCES values(213,'Tulu', 'M',23,25,158,164)
insert into PREFERENCES values(214,'Hindi', 'M',23,27,164,184)
/*These two inserts will be used for demo*/
	
insert into PREFERENCES values(215,'Tulu', 'F',27,31,166,188)
insert into PREFERENCES values(216,'Hindi', 'F',23,32,182,190)

select * from PREFERENCES;


create table MATCHES_(Match_id int primary key IDENTITY(101,1) ,
Initiator_id int ,
Reciever_id int,
constraint fk2 foreign key (Initiator_id) references USER_ACCOUNT(Userid), 
constraint fk3 foreign key (Reciever_id) references USER_ACCOUNT(Userid))

create table BLOCKED_PROFILES(Userid int references USER_ACCOUNT(userid), 
Blocked_id int references USER_ACCOUNT(userid),
primary key(userid,Blocked_id))






insert into BLOCKED_PROFILES values(201, 206)
insert into BLOCKED_PROFILES values(202, 207)
insert into BLOCKED_PROFILES values(203, 201)
insert into BLOCKED_PROFILES values(204, 202)
insert into BLOCKED_PROFILES values(205, 209)
insert into BLOCKED_PROFILES values(205, 203)
insert into BLOCKED_PROFILES values(205, 202)
insert into BLOCKED_PROFILES values(202, 209)

select * from BLOCKED_PROFILES

--BASIC QUERIES


/*Get the name of the person who would like to be in a relationship with a person who's mother tongue is Hindi*/
select Fname,Lname
from USER_ACCOUNT U, PREFERENCES P
where U.Userid=P.Userid and U.Mother_Tongue='Hindi'


/*Get the names of users who have blocked atleast two other users*/

create view v2 as 
select U.Fname,U.Lname,count(*) as No_of_blocks
from BLOCKED_PROFILES B, USER_ACCOUNT U
where B.Userid=U.Userid
group by U.Fname,U.Lname
having count(*)>=2

select Fname,Lname
from v2



--03.
/*Get the count of number of matches per user*/

select Fname,Lname,Num_of_matches
from USER_ACCOUNT


/*STORED PROCEDURE*/

CREATE PROCEDURE p2 @MotherTongue varchar(20), @Sex char(5)
AS
select *
from USER_ACCOUNT
where Gender=@Sex and Mother_Tongue=@MotherTongue;

EXEC p2 @Sex='M', @MotherTongue='Kannada';


/*TRIGGER STATEMENTS*/

/*Trigger 1 creates a view which contains the Initiator and Reciever Ids depending on the preferences of the user*/
	
create trigger t1
on PREFERENCES
after insert
as

declare @sql nvarchar(max) = 
    N'use [matrimonydbs]; 
      exec (''create view v3 as
	select P.Userid as Initiator_id, U.Userid as Reciever_id
	from USER_ACCOUNT U, PREFERENCES P
	where P.Mother_Tongue=U.Mother_Tongue and P.Gender=U.Gender and 
	U.Age>=P.Min_Age and U.Age<=P.Max_Age and U.Height_in_cm>=P.Min_Height and
	U.Height_in_cm<=P.Max_Height and P.Userid=(SELECT TOP 1 Userid FROM PREFERENCES ORDER BY userid DESC)
'')';	


--Trigger with manual insertion of PREFERENCES Entity

create trigger t1
on PREFERENCES
after insert
as
	update USER_ACCOUNT
	set Num_of_matches=(select count(*) as No_of_matches
	from USER_ACCOUNT U, PREFERENCES P
	where P.Mother_Tongue=U.Mother_Tongue and
	P.Gender=U.Gender and U.Age>=P.Min_Age and
	U.Age<=P.Max_Age and U.Height_in_cm>=P.Min_Height and
	U.Height_in_cm<=P.Max_Height and P.Userid=(SELECT TOP 1 Userid FROM PREFERENCES ORDER BY userid DESC)
	group by P.Userid)
	where Userid=(SELECT TOP 1 Userid FROM PREFERENCES ORDER BY userid DESC)


/*The view created in trigger 1 is inserted into the MATCHES_ table in trigger 2 along with a delay of 4 seconds and this trigger also updates the 
No_of_matches column in USER_ACCOUNT table*/

create trigger t2
on PREFERENCES
after insert
as
	WAITFOR DELAY '00:00:04';
	INSERT INTO MATCHES_ (Initiator_id, Reciever_id)
	SELECT Initiator_id, Reciever_id FROM v3;

	update USER_ACCOUNT
	set Num_of_matches=(select count(*) as No_of_matches
	from USER_ACCOUNT U, PREFERENCES P
	where P.Mother_Tongue=U.Mother_Tongue and P.Gender=U.Gender and U.Age>=P.Min_Age and U.Age<=P.Max_Age and U.Height_in_cm>=P.Min_Height and U.Height_in_cm<=P.Max_Height and P.Userid=(SELECT TOP 1 Userid FROM PREFERENCES ORDER BY userid DESC)
	group by P.Userid)
	where Userid=(SELECT TOP 1 Userid FROM PREFERENCES ORDER BY userid DESC)

/*Trigger 3 is used to drop the view*/

create trigger t3
on PREFERENCES
after insert
as
	WAITFOR DELAY '00:00:05';
	drop view v3




/*Select Statements*/
SELECT * FROM USER_ACCOUNT
SELECT * FROM PREFERENCES
SELECT * FROM MATCHES_
SELECT * FROM BLOCKED_PROFILES



/*Drop Statements*/
drop table USER_ACCOUNT
drop table PREFERENCES
drop table MATCHES_
drop table BLOCKED_PROFILES







drop trigger t2;
drop trigger t1;
drop trigger t3;
