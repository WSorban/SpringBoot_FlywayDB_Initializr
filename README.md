# SpringBoot_FlywayDB_Initializr

Steps to initialize FlywayDB, and run it:

NOTE: You will have to follow these steps for each new application profile you make, on an existing Schema

1. Take the application and import it as a Maven project
2. Clean, Install
3. Since Flyway has been initialized on an existing schema, we must create a first version. 
To do that, we must create the VERSION table, and make one entry in it.
Therefore, run the following querries, manually, in your database:

CREATE TABLE VERSION (
	"version_rank" INTEGER NOT NULL,
	"installed_rank" INTEGER NOT NULL,
	"version" VARCHAR(50) NOT NULL,
	"description" VARCHAR(200) NOT NULL,
	"type" VARCHAR(20) NOT NULL,
	"script" VARCHAR(1000) NOT NULL,
	"checksum" INTEGER,
	"installed_by" VARCHAR(100) NOT NULL,
	"installed_on" TIMESTAMP NOT NULL DEFAULT CURRENT TIMESTAMP,
	"execution_time" INTEGER NOT NULL,
	"success" SMALLINT NOT NULL,
	CONSTRAINT "VERSION_pk" PRIMARY KEY ("version")
) ;
CREATE INDEX "VERSION_ir_idx" ON VERSION ("installed_rank") ;
CREATE INDEX "VERSION_s_idx" ON VERSION ("success") ;
CREATE INDEX "VERSION_vr_idx" ON VERSION ("version_rank") ;

4. You may notice that the src/main/resources/sql/test3 folder contains a V1__test.sql
You must make sure not to make any changes to this file, as FlywayDB is doing a checksum on it, and compares it to the entry in the VERSION table.
5. We have not yet made the entry in the VERSION table, for version 1. So run the following query:

INSERT INTO VERSION ("version_rank","installed_rank","version","description","type","script","checksum","installed_by","installed_on","execution_time","success") VALUES (
1,1,'1','test','SQL','V1__test.sql',863939790,'DB2INST7','2017-09-27 13:54:21.858',197,1);

6. Now, all the future sql files in the SQL/test3 folder will be taken by FlywayDB, and migrated.
7. A new entry will be created for each SQL file, in the VERSION table.

NOTE: These steps are described for the application profile test3.
If you want another/new profile, you must take the "V1_" file from the src/main/resources/sql/[profile_name] folder, and place it in a the new [profile_name] folder

Notes:

1. FlywayDB has been integrated into a SpringBoot application.
2. This allows us to have multiple profiles, for multiple migration scripts/folders, and multiple databases.
3. The property files must all be located in the src/main/resources folder
4. The property files must always follow the following format: application-[profile_name].properties
5. To initialize one profile, or another, you must add the following argument in your run configurations: -Dspring.profiles.active=[profile_name]
6. The property files must contain the following information: Datasource (URL, credentials), Flyway properties
7. The following properties are set in the demo application:

#enable/disable flywayDB
flyway.enabled=true

#Set the location of the SQL files. Ideally, each application profile should have a different folder inside the SQL folder, named by the profile name
flyway.locations=classpath:sql/test3

#Set the name of the version table
flyway.table = VERSION

#Set the prefix that each SQL file will have, so Flyway looks for those, and finds them
flyway.sql-migration-prefix=V

#Set the extension that each SQL file will have, so Flyway looks for those, and finds them
flyway.sql-migration-suffix=.sql

#If this is set to true, then checksum validations will be performed, and the application will fail to launch, if checksum discrepancies are found (in case clean-on-validation-error is set to false)
flyway.validate-on-migrate=true

#If this is set to true, if a checksum discrepancy is found (SQL checksum vs checmsum stored in the VERSION table), then all tables will be dropped, the application will restart from version 1; If we have an already existing table, and do not have versioned SQL file for them, then make sure to NEVER set this option to TRUE
flyway.clean-on-validation-error=false

8. Depending on which Database vendor you are using, you must import the respective dependencies, from the POM file.
9. Right now, DB2 dependencies are set.
