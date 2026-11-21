# My-first-project
Project 1
CREATE DEFINER=`biswajit.nayak`@`%` PROCEDURE `USP_PC_PATIENT_REGISTRATION`(IN `p_patientRegistrationId` INT, IN `p_uprnNo` VARCHAR(45), IN `p_dateOfRegd` TIMESTAMP(6), IN `p_firstName` VARCHAR(45), IN `p_middleName` VARCHAR(45), IN `p_lastName` VARCHAR(45), IN `p_mobileNo` VARCHAR(45), IN `p_mobileNoValidate` CHAR(1), IN `p_patientPhoto` VARCHAR(45), IN `p_identyTypeId` INT, IN `p_identyCardNo` VARCHAR(45), IN `p_address` VARCHAR(500), IN `p_stateId` INT, IN `p_pin` INT, IN `p_genderId` INT, IN `p_maritalStatusId` INT, IN `p_dateOfBirth` DATE, IN `p_bloodGroupId` INT, IN `p_bloodGroupValidated` CHAR(1), IN `p_bloodGroupUpload` VARCHAR(45), IN `p_bSKYNum` VARCHAR(45), IN `p_ABHANum` VARCHAR(45), IN `p_bPLNum` VARCHAR(45), IN `p_rationCardNum` VARCHAR(45), IN `p_passportNum` VARCHAR(45), IN `p_pANNo` VARCHAR(45), IN `p_visaNo` VARCHAR(45), IN `p_occupation` VARCHAR(45), IN `p_annualIncome` INT, IN `p_religionId` INT, IN `p_insuranceIfAny` CHAR(1), IN `p_insuranceCardNo` VARCHAR(45), IN `p_patientSubTypeId` INT, IN `p_insuranceExpiryDate` DATE, IN `p_otherInsuranceCompany` VARCHAR(100), IN `p_otherDetails` VARCHAR(45), IN `p_currentAddress` VARCHAR(500), IN `p_contactNameInCaseOfEmergency` VARCHAR(45), IN `p_alternateContactNum` VARCHAR(45), IN `p_relationshipWithContactPerson` VARCHAR(45), IN `p_emailId` VARCHAR(45), IN `p_contactNum` VARCHAR(45), IN `p_heriditaryDiseasesIfAny` CHAR(1), IN `p_heriditaryDiseases` VARCHAR(100), IN `p_seriousAilmentIfAny` CHAR(1), IN `p_seriousAilment` VARCHAR(100), IN `p_otherDieseasesIfAny` CHAR(1), IN `p_otherDieseases` VARCHAR(100), IN `p_anySurgeryDone` CHAR(1), IN `p_SurgeryDone` VARCHAR(100), IN `p_masterCardPrint` CHAR(1), IN `p_invoiceRaised` CHAR(1), IN `p_collection` CHAR(1), IN `p_businessAreaId` INT, IN `p_createdBy` INT, IN `p_updatedBy` INT, IN `p_status` BIT, IN `p_ACTIONFLAG` VARCHAR(50), OUT `p_yes_no` VARCHAR(200), IN `p_localarea` VARCHAR(45), IN `p_district` VARCHAR(45), IN `p_patientTempName` VARCHAR(55), IN `p_pageno` INT, IN `p_perpage` INT, IN `p_dateOfRegdExistingSystem` TIMESTAMP(6), IN `p_fathersName` VARCHAR(45), IN `p_deathCertificate` VARCHAR(500), IN `p_deathDate` DATETIME, IN `p_EmpNo` VARCHAR(45), IN `p_deathRemark` VARCHAR(500), IN `p_AadharNo` VARCHAR(16), IN `p_mothersName` VARCHAR(45),IN `p_uhidStatus` INT
,IN p_age INT,IN p_gender VARCHAR(20))
BEGIN
/*
Modified By:Biswajit Nayak
Created On:22-04-2026
* Variable Declarations Changed
* Flags:SEARCHCOUNTER,SEARCHPATIENTDETAILS,ACTIVEINACTIVE,UPDATE_BLOODGROUP,UPDATE,ADD: Query Optimized. Indexation Applied.
*/  
DECLARE qryExistQuery,qryNewQuery,qryConditionQuery longtext;
DECLARE v_Year,v_Month,v_Days,p_maxUserID,v_error,v_intCount,v_himsUniqueRequestId,i,flfMobNoCk,flmMobNoCk,flMobNoCk,adharNoCk,adharCk,eduID,empcount,ipdcount,opdcount,lastInsertId,COUNTER INTEGER DEFAULT 0;
DECLARE v_Age,v_FullName,v_Uprn,v_himsBillId VARCHAR(45);
DECLARE CONTINUE HANDLER FOR NOT FOUND SET v_error = 1;
DECLARE EXIT HANDLER FOR SQLEXCEPTION  

BEGIN 
GET DIAGNOSTICS CONDITION 1 
@sqlstate = RETURNED_SQLSTATE,  
@errno = MYSQL_ERRNO, 
@text = MESSAGE_TEXT;   
ROLLBACK; 
set @blockName='USP_PC_PATIENT_REGISTRATION';	
set @actionCode=p_ACTIONFLAG;			
insert into sys_procedure_exception_log(state,errorNumber,error,procedureName,actionCode,exceptionDateTime) 
values(@sqlstate, @errno,concat(@text,' * Input Parameter List - ( patientRegistration Id :-',COALESCE(p_patientRegistrationId,0),', uprn No :-',COALESCE(p_uprnNo,0)),@blockName,@actionCode,CURRENT_TIMESTAMP(3)); 
END;
/*
IF P_ACTIONFLAG = 'ADD' THEN
start transaction;
SELECT COUNT(CASE WHEN TRIM(AadharNo) = TRIM(p_AadharNo) AND status = 0 THEN 1 END) AS adharCk,
COUNT(CASE WHEN TRIM(AadharNo) = TRIM(p_AadharNo) AND TRIM(mobileNo) = TRIM(p_mobileNo) AND status = 0 THEN 1 END) AS adharNoCk,
COUNT(CASE WHEN TRIM(UPPER(firstName)) = TRIM(UPPER(p_firstName))AND TRIM(UPPER(middleName)) = TRIM(UPPER(p_middleName))AND TRIM(UPPER(lastName)) = TRIM(UPPER(p_lastName))AND TRIM(mobileNo) = TRIM(p_mobileNo)AND status = 0 THEN 1 END) AS flMobNoCk,
COUNT(CASE WHEN TRIM(UPPER(firstName)) = TRIM(UPPER(p_firstName))AND TRIM(UPPER(middleName)) = TRIM(UPPER(p_middleName))AND TRIM(UPPER(lastName)) = TRIM(UPPER(p_lastName))AND TRIM(UPPER(mothersName)) = TRIM(UPPER(p_mothersName))AND TRIM(mobileNo) = TRIM(p_mobileNo)AND status = 0 THEN 1 END) AS flmMobNoCk,
COUNT(CASE WHEN TRIM(UPPER(firstName)) = TRIM(UPPER(p_firstName))AND TRIM(UPPER(middleName)) = TRIM(UPPER(p_middleName))AND TRIM(UPPER(lastName)) = TRIM(UPPER(p_lastName))AND TRIM(UPPER(fathersName)) = TRIM(UPPER(p_fathersName))AND TRIM(mobileNo) = TRIM(p_mobileNo)AND status = 0 THEN 1 END) AS flfMobNoCk
INTO adharCk,adharNoCk,flMobNoCk,flmMobNoCk,flfMobNoCk FROM pc_patient_registration;

if (adharCk>=1 or adharNoCk>=1 or flMobNoCk>=1 or flmMobNoCk>=1 or flfMobNoCk>=1)
then select 'Patient Already Exists' into p_yes_no;
ELSE
SET v_Uprn=(SELECT CONCAT((SELECT aliasName FROM admin_business_area WHERE businessAreaId = p_businessAreaId),'-', LPAD( (SELECT IFNULL(MAX(CAST(SUBSTRING_INDEX(uprnNo, '-', -1) AS UNSIGNED)), 0) + 1 FROM pc_patient_registration), 8, '0')));
INSERT INTO `pc_patient_registration`(`uprnNo`,`dateOfRegd`,`firstName`,`middleName`,`lastName`,`mobileNo`,`mobileNoValidate`,`patientPhoto`,`identyTypeId`,`identyCardNo`,`AadharNo`,`address`,`stateId`,`pin`,`genderId`,`maritalStatusId`,`dateOfBirth`,`bloodGroupId`,`bloodGroupValidated`,`bloodGroupUpload`,`bSKYNum`,`ABHANum`,`bPLNum`,`rationCardNum`,`passportNum`,`pANNo`,`visaNo`,`occupation`,`annualIncome`,`religionId`,`insuranceIfAny`,`insuranceCardNo`,`patientSubTypeId`,`insuranceExpiryDate`,`otherInsuranceCompany`,`otherDetails`,`currentAddress`,`contactNameInCaseOfEmergency`,`alternateContactNum`,`relationshipWithContactPerson`,`emailId`,`contactNum`,`heriditaryDiseasesIfAny`,`heriditaryDiseases`,`seriousAilmentIfAny`,`seriousAilment`,`otherDieseasesIfAny`,`otherDieseases`,`anySurgeryDone`,`SurgeryDone`,`masterCardPrint`,`invoiceRaised`,`collection`,`businessAreaId`, `createdBy`,`createdOn`,`status`,`localarea`,`district`,`patientTempName`,`dateOfRegdExistingSystem`,`fathersName`,`mothersName`)
VALUES(v_Uprn,CURRENT_TIMESTAMP,p_firstName,p_middleName,p_lastName,p_mobileNo,p_mobileNoValidate,p_patientPhoto,p_identyTypeId,p_identyCardNo,p_AadharNo,p_address,p_stateId,p_pin,p_genderId,p_maritalStatusId,p_dateOfBirth,p_bloodGroupId,p_bloodGroupValidated,p_bloodGroupUpload,p_bSKYNum,p_ABHANum,p_bPLNum,p_rationCardNum,p_passportNum,p_pANNo,p_visaNo,p_occupation,p_annualIncome,p_religionId,p_insuranceIfAny,p_insuranceCardNo,p_patientSubTypeId,p_insuranceExpiryDate,p_otherInsuranceCompany,p_otherDetails,p_currentAddress,p_contactNameInCaseOfEmergency,p_alternateContactNum,p_relationshipWithContactPerson,p_emailId,p_contactNum,p_heriditaryDiseasesIfAny,p_heriditaryDiseases,p_seriousAilmentIfAny,p_seriousAilment,p_otherDieseasesIfAny,p_otherDieseases,p_anySurgeryDone,p_SurgeryDone,p_masterCardPrint,p_invoiceRaised,p_collection,p_businessAreaId,p_createdBy,CURRENT_TIMESTAMP,0,p_localarea,p_district,p_patientTempName,CURRENT_TIMESTAMP,p_fathersName,p_mothersName);  
commit;
SELECT LAST_INSERT_ID() into lastInsertId; 
IF lastInsertId  >= 1 THEN
SELECT lastInsertId into p_yes_no;
else 
SELECT "Please Enter correct value" into p_yes_no;
END IF;
END IF;
END IF;

IF P_ACTIONFLAG = 'UPDATE' THEN
start transaction;
SELECT COUNT(CASE WHEN TRIM(AadharNo) = TRIM(p_AadharNo) AND status = 0 THEN 1 END) AS adharCk,
COUNT(CASE WHEN TRIM(AadharNo) = TRIM(p_AadharNo) AND TRIM(mobileNo) = TRIM(p_mobileNo) AND status = 0 THEN 1 END) AS adharNoCk,
COUNT(CASE WHEN TRIM(UPPER(firstName)) = TRIM(UPPER(p_firstName))AND TRIM(UPPER(middleName)) = TRIM(UPPER(p_middleName))AND TRIM(UPPER(lastName)) = TRIM(UPPER(p_lastName))AND TRIM(mobileNo) = TRIM(p_mobileNo)AND status = 0 THEN 1 END) AS flMobNoCk,
COUNT(CASE WHEN TRIM(UPPER(firstName)) = TRIM(UPPER(p_firstName))AND TRIM(UPPER(middleName)) = TRIM(UPPER(p_middleName))AND TRIM(UPPER(lastName)) = TRIM(UPPER(p_lastName))AND TRIM(UPPER(mothersName)) = TRIM(UPPER(p_mothersName))AND TRIM(mobileNo) = TRIM(p_mobileNo)AND status = 0 THEN 1 END) AS flmMobNoCk,
COUNT(CASE WHEN TRIM(UPPER(firstName)) = TRIM(UPPER(p_firstName))AND TRIM(UPPER(middleName)) = TRIM(UPPER(p_middleName))AND TRIM(UPPER(lastName)) = TRIM(UPPER(p_lastName))AND TRIM(UPPER(fathersName)) = TRIM(UPPER(p_fathersName))AND TRIM(mobileNo) = TRIM(p_mobileNo)AND status = 0 THEN 1 END) AS flfMobNoCk
INTO adharCk,adharNoCk,flMobNoCk,flmMobNoCk,flfMobNoCk FROM pc_patient_registration where patientRegistrationId <>p_patientRegistrationId;

if (adharCk>=1 or adharNoCk>=1 or flMobNoCk>=1 or flmMobNoCk>=1 or flfMobNoCk>=1)
then select 'Patient Already Exists' into p_yes_no;
ELSE   
UPDATE pc_patient_registration ppr 
SET ppr.firstName=p_firstName,ppr.middleName=p_middleName,ppr.lastName=p_lastName,ppr.mobileNo=p_mobileNo,ppr.patientPhoto=p_patientPhoto,ppr.identyTypeId=p_identyTypeId,ppr.identyCardNo=p_identyCardNo,
ppr.AadharNo = p_AadharNo,ppr.address=p_address,ppr.stateId=p_stateId,ppr.pin=p_pin,ppr.maritalStatusId=p_maritalStatusId,ppr.dateOfBirth=p_dateOfBirth,ppr.bloodGroupId=p_bloodGroupId,
ppr.bloodGroupValidated=p_bloodGroupValidated,ppr.bloodGroupUpload=p_bloodGroupUpload,ppr.bSKYNum=p_bSKYNum,ppr.bPLNum=p_bPLNum,ppr.rationCardNum=p_rationCardNum,ppr.passportNum=p_passportNum,
ppr.pANNo=p_pANNo,ppr.visaNo=p_visaNo,ppr.occupation=p_occupation,ppr.religionId=p_religionId,ppr.insuranceIfAny=p_insuranceIfAny,ppr.insuranceCardNo=p_insuranceCardNo,ppr.patientSubTypeId=p_patientSubTypeId,
ppr.insuranceExpiryDate=p_insuranceExpiryDate,ppr.otherInsuranceCompany=p_otherInsuranceCompany,ppr.otherDetails=p_otherDetails,ppr.currentAddress=p_currentAddress,ppr.contactNameInCaseOfEmergency=p_contactNameInCaseOfEmergency,
ppr.alternateContactNum=p_alternateContactNum,ppr.relationshipWithContactPerson=p_relationshipWithContactPerson,ppr.emailId=p_emailId,ppr.contactNum=p_contactNum,ppr.heriditaryDiseasesIfAny=p_heriditaryDiseasesIfAny,
ppr.heriditaryDiseases=p_heriditaryDiseases,ppr.seriousAilmentIfAny=p_seriousAilmentIfAny,ppr.seriousAilment=p_seriousAilment,ppr.otherDieseasesIfAny=p_otherDieseasesIfAny,ppr.otherDieseases=p_otherDieseases,
ppr.anySurgeryDone=p_anySurgeryDone,ppr.SurgeryDone=p_SurgeryDone,ppr.masterCardPrint=p_masterCardPrint,ppr.invoiceRaised=p_invoiceRaised,ppr.collection=p_collection,ppr.businessAreaId=p_businessAreaId,ppr.updatedOn=CURRENT_TIMESTAMP,
ppr.updatedBy=p_updatedBy,ppr.localarea=p_localarea,ppr.district=p_district,ppr.dateOfRegdExistingSystem=CURRENT_TIMESTAMP,ppr.fathersName=p_fathersName,ppr.mothersName=p_mothersName
WHERE ppr.patientRegistrationId = p_patientRegistrationId AND ppr.status = 0 ;
SET lastInsertId = ROW_COUNT();
COMMIT;
IF lastInsertId > 0 THEN SELECT "Updated Successfully" into p_yes_no;
ELSE 
SELECT "Value not updated" into p_yes_no;
END IF;
END IF;
END IF;
*/
IF P_ACTIONFLAG = 'UPDATE_BLOODGROUP' THEN
start transaction;
UPDATE pc_patient_registration SET bloodGroupId=p_bloodGroupId,bloodGroupValidated=p_bloodGroupValidated,bloodGroupUpload=p_bloodGroupUpload  WHERE patientRegistrationId=p_patientRegistrationId AND status=0;
commit;
SELECT COUNT(patientRegistrationId) INTO p_yes_no from pc_patient_registration WHERE patientRegistrationId=p_patientRegistrationId and  bloodGroupId=p_bloodGroupId AND status=0;
IF p_yes_no  >= 1 THEN SELECT "Blood Group Updated Successfully" into p_yes_no;
ELSE 
SELECT "Value not updated" into p_yes_no;
END IF;
END IF;
   
IF P_ACTIONFLAG = 'VIEW' THEN
SET v_himsUniqueRequestId = RIGHT(UNIX_TIMESTAMP(), 6);
SET v_himsBillId= ((SELECT CONCAT(billType, billId) FROM dothealth_api_log WHERE uprnNo = (SELECT uprnNo FROM pc_patient_registration WHERE patientRegistrationId = p_patientRegistrationId
AND status = 0) ORDER BY apilogId DESC LIMIT 1));
SELECT v_himsUniqueRequestId HimsUniqueRequestId,v_himsBillId HimsBillId,PR.patientRegistrationId,PR.uprnNo,SUBSTRING_INDEX(PR.uprnNo, '-', -1) as UHID_PART,PR.dateOfRegd,PR.firstName,PR.middleName,PR.createdBy,
	PR.lastName,PR.mobileNo,PR.mobileNoValidate,PR.patientPhoto,PR.identyTypeId,PR.identyCardNo,PR.ABHANum,PR.otherInsuranceCompany,PR.address,PR.stateId,(case when PR.pin=0 then null ELSE  PR.pin end) pin,PR.genderId,PR.maritalStatusId,PR.dateOfBirth,PR.bloodGroupId,PR.bloodGroupValidated,PR.bloodGroupUpload,PR.bSKYNum,PR.bPLNum,PR.rationCardNum,PR.passportNum,PR.pANNo,PR.visaNo,PR.occupation,(case when PR.annualIncome=0 then null else PR.annualIncome end) annualIncome,
    PR.religionId,PR.insuranceIfAny,PR.insuranceCardNo,PR.patientSubTypeId,PR.insuranceExpiryDate,PR.otherDetails,PR.currentAddress,PR.contactNameInCaseOfEmergency,PR.alternateContactNum,PR.relationshipWithContactPerson,PR.emailId,PR.contactNum,PR.heriditaryDiseasesIfAny,PR.heriditaryDiseases,PR.seriousAilmentIfAny,BG.bloodGroup,
	PR.seriousAilment,PR.otherDieseasesIfAny,PR.otherDieseases,PR.anySurgeryDone,PR.SurgeryDone,PR.masterCardPrint,PR.invoiceRaised,SLI.userFullName  CREATEDBY,PR.localarea,PR.district,PR.collection,PR.businessAreaId ,SI.identyType,SS.stateName,SG.gender,MS.maritalStatus,SR.religion,BS.businessUnitName,
   CASE
    WHEN DATE(PR.dateOfBirth) = CURDATE() THEN '0 Y 0 M 1 D'
    ELSE CONCAT(
        TIMESTAMPDIFF(YEAR, PR.dateOfBirth, CURDATE()), ' Y ',
        MOD(TIMESTAMPDIFF(MONTH, PR.dateOfBirth, CURDATE()), 12), ' M ',
        GREATEST(
            1,
            DATEDIFF(
                CURDATE(),
                DATE_ADD(
                    DATE_ADD(
                        PR.dateOfBirth,
                        INTERVAL TIMESTAMPDIFF(YEAR, PR.dateOfBirth, CURDATE()) YEAR
                    ),
                    INTERVAL MOD(TIMESTAMPDIFF(MONTH, PR.dateOfBirth, CURDATE()), 12) MONTH
                )
            )
        ),
        ' D'
    )
END AS age,PS.patientSubType,PR.patientTempName,PR.dateOfRegdExistingSystem,PR.fathersName,PR.AadharNo,PR.mothersName
	FROM pc_patient_registration PR
	LEFT JOIN sys_blood_group BG ON BG.bloodGroupId=PR.bloodGroupId 
	LEFT JOIN sys_identy_type SI on SI.identyTypeId=PR.identyTypeId
	LEFT JOIN sys_state SS on SS.stateId=PR.stateId
	LEFT JOIN sys_gender SG on SG.genderId=PR.genderId
	LEFT JOIN sys_marital_status MS on MS.maritalStatusId=PR.maritalStatusId
	LEFT JOIN sys_religion SR on SR.religionId=PR.religionId
	LEFT JOIN sys_patient_sub_type PS on PS.patientSubTypeId=PR.patientSubTypeId
	LEFT JOIN admin_business_area BS on BS.businessAreaId=PR.businessAreaId
	LEFT JOIN sys_login_information SLI ON SLI.userId=PR.createdBy
	WHERE   PR.patientRegistrationId=p_patientRegistrationId  and PR.status=0 order by  PR.patientRegistrationId DESC;
END IF;
/*
IF P_ACTIONFLAG = 'VIEWALL' THEN
     SELECT PR.patientRegistrationId,PR.uprnNo,PR.dateOfRegd,concat(PR.firstName,' ',PR.middleName,' ',PR.lastName) AS PatientNam,PR.firstName,PR.middleName,
	PR.lastName,PR.mobileNo,PR.mobileNoValidate,PR.patientPhoto,PR.identyTypeId,PR.identyCardNo,PR.ABHANum,PR.otherInsuranceCompany,
	PR.address,PR.stateId,PR.pin,PR.genderId,PR.maritalStatusId,PR.dateOfBirth,PR.bloodGroupId,PR.bloodGroupValidated,PR.bloodGroupUpload,PR.bSKYNum,
	PR.bPLNum,PR.rationCardNum,PR.passportNum,PR.pANNo,PR.visaNo,PR.occupation,PR.annualIncome,PR.religionId,PR.insuranceIfAny,PR.insuranceCardNo,
	PR.patientSubTypeId,PR.insuranceExpiryDate,PR.otherDetails,PR.currentAddress,PR.contactNameInCaseOfEmergency,PR.alternateContactNum,
	PR.relationshipWithContactPerson,PR.emailId,PR.contactNum,PR.heriditaryDiseasesIfAny,PR.heriditaryDiseases,PR.seriousAilmentIfAny,
	PR.seriousAilment,PR.otherDieseasesIfAny,PR.otherDieseases,PR.anySurgeryDone,PR.SurgeryDone,PR.masterCardPrint,PR.invoiceRaised,
	PR.collection,PR.businessAreaId,SS.stateName,SG.gender,MS.maritalStatus,SR.religion,BS.businessUnitName,PR.status,PR.localarea,PR.district,PR.dateOfRegdExistingSystem,PR.fathersName,PR.AadharNo,PR.mothersName
	FROM pc_patient_registration PR
	LEFT join sys_state SS on SS.stateId=PR.stateId
	INNER JOIN sys_gender SG on SG.genderId=PR.genderId
	LEFT OUTER JOIN sys_marital_status MS on MS.maritalStatusId=PR.maritalStatusId
	LEFT OUTER JOIN sys_religion SR on SR.religionId=PR.religionId
	INNER JOIN admin_business_area as BS on BS.businessAreaId=PR.businessAreaId
	WHERE DATE(PR.createdOn) = CURRENT_DATE order by  PR.patientRegistrationId DESC;
END IF;
*/
IF P_ACTIONFLAG = 'GETCOUNT' THEN
    select count(patientRegistrationId) INTO p_yes_no from pc_patient_registration;
END IF;
IF P_ACTIONFLAG = 'ACTIVEINACTIVE' THEN
IF p_status = 1 THEN
SET opdcount = (SELECT COUNT(patientTicketRegistrationOpdId) FROM pc_patient_ticket_registration_opd WHERE patientRegistrationId = p_patientRegistrationId AND drCheckOutStatus = 'N');
SET ipdcount = (SELECT COUNT(ipnRegistrationId) FROM pc_ipn_registration WHERE patientRegistrationId = p_patientRegistrationId AND ipnChekoutstatus = 'N');
SET empcount = (SELECT COUNT(patientTicketRegistrationEMPNId) FROM pc_patient_ticket_registration_empn WHERE patientRegistrationId = p_patientRegistrationId AND drCheckOutStatus = 'N');
IF (opdcount >= 1 OR ipdcount >= 1 OR empcount >= 1) THEN 
SET p_yes_no = 'Patient treatment is going on OPD/IPD/EMERGENCY';
ELSE
UPDATE pc_patient_registration SET status = p_status WHERE patientRegistrationId = p_patientRegistrationId;
SELECT ROW_COUNT() INTO lastInsertId;
IF lastInsertId = 1 THEN SET p_yes_no = 'Updated Successfully';
ELSE 
SET p_yes_no = 'Value not updated';
END IF;
END IF;
ELSE
UPDATE pc_patient_registration SET status = p_status WHERE patientRegistrationId = p_patientRegistrationId;
SELECT ROW_COUNT() INTO lastInsertId;
IF lastInsertId = 1 THEN SET p_yes_no = 'Updated Successfully';
ELSE 
SET p_yes_no = 'Value not updated';
END IF;
END IF;
END IF;

/*
IF P_ACTIONFLAG = 'SEARCHPATIENTDETAILS' THEN
IF p_perpage=0 THEN 
SELECT p.patientRegistrationId,p.uprnNo,p.dateOfRegd,p.dateOfRegdExistingSystem,TRIM(REPLACE(CONCAT(p.firstName, CASE WHEN p.middleName <> '' THEN CONCAT(' ', p.middleName) ELSE '' END,' ',p.lastName),' ', ' ')) PatientNam,
p.mobileNo,p.AadharNo,p.fathersName,s.stateName,p.status,p.mothersName,p.firstName,p.middleName,p.lastName, p.mobileNoValidate,p.patientPhoto,p.identyTypeId,p.identyCardNo,p.ABHANum,p.otherInsuranceCompany,
p.address,p.stateId,p.pin,p.genderId,p.maritalStatusId,p.dateOfBirth,p.bloodGroupId,p.bloodGroupValidated,p.bloodGroupUpload,p.bSKYNum,p.bPLNum,p.rationCardNum,p.passportNum,p.pANNo,p.visaNo,p.occupation,p.annualIncome,p.religionId,p.insuranceIfAny,p.insuranceCardNo,p.patientSubTypeId,p.insuranceExpiryDate,p.otherDetails,p.currentAddress,p.contactNameInCaseOfEmergency,p.alternateContactNum,
p.relationshipWithContactPerson,p.emailId,p.contactNum,p.heriditaryDiseasesIfAny,p.heriditaryDiseases,p.seriousAilmentIfAny,p.seriousAilment,p.otherDieseasesIfAny,p.otherDieseases,p.anySurgeryDone,p.SurgeryDone,p.masterCardPrint,p.invoiceRaised,p.collection,p.businessAreaId,p.localarea,p.district,SG.gender,MS.maritalStatus,SR.religion,BS.businessUnitName,
count(1) over() COUNTER
FROM 
pc_patient_registration p
LEFT JOIN sys_state s ON s.stateId=p.stateId
INNER JOIN sys_gender as SG on SG.genderId=p.genderId
LEFT OUTER JOIN sys_marital_status as MS on MS.maritalStatusId=p.maritalStatusId
LEFT OUTER JOIN sys_religion as SR on SR.religionId=p.religionId
INNER JOIN admin_business_area as BS on BS.businessAreaId=p.businessAreaId
 WHERE (DATE(p.createdOn)>=DATE(p_dateOfRegdExistingSystem) OR p_dateOfRegdExistingSystem IS NULL) AND (DATE(p.createdOn)<=DATE(p_dateOfRegd) OR p_dateOfRegd IS NULL)
-- AND p.businessAreaId=p_businessAreaId
AND (trim(p.uprnNo)=trim(p_uprnNo) OR p_uprnNo IS NULL)
AND (trim(p.AadharNo)=trim(p_AadharNo) OR p_AadharNo IS NULL)
AND (trim(p.mobileNo)=trim(p_mobileNo) OR p_mobileNo IS NULL)
AND (p.createdBy=p_createdBy OR p_createdBy IS NULL)
AND (p.stateId=p_stateId OR p_stateId =0 OR p_stateId IS NULL)
AND (TRIM(REPLACE(CONCAT(p.firstName,CASE WHEN p.middleName <> '' THEN CONCAT(' ', p.middleName) ELSE '' END,' ',p.lastName),'  ',' ')) LIKE CONCAT('%', TRIM(p_firstName), '%')  OR p_firstName IS NULL)
ORDER BY p.patientRegistrationId DESC;
ELSE 
SELECT p.patientRegistrationId,p.uprnNo,p.dateOfRegd,p.dateOfRegdExistingSystem,TRIM(REPLACE(CONCAT(p.firstName, CASE WHEN p.middleName <> '' THEN CONCAT(' ', p.middleName) ELSE '' END,' ',p.lastName),' ', ' ')) PatientNam,
p.mobileNo,p.AadharNo,p.fathersName,s.stateName,p.status,p.mothersName,p.firstName,p.middleName,p.lastName, p.mobileNoValidate,p.patientPhoto,p.identyTypeId,p.identyCardNo,p.ABHANum,p.otherInsuranceCompany,
p.address,p.stateId,p.pin,p.genderId,p.maritalStatusId,p.dateOfBirth,p.bloodGroupId,p.bloodGroupValidated,p.bloodGroupUpload,p.bSKYNum,p.bPLNum,p.rationCardNum,p.passportNum,p.pANNo,p.visaNo,p.occupation,p.annualIncome,p.religionId,p.insuranceIfAny,p.insuranceCardNo,p.patientSubTypeId,p.insuranceExpiryDate,p.otherDetails,p.currentAddress,p.contactNameInCaseOfEmergency,p.alternateContactNum,
p.relationshipWithContactPerson,p.emailId,p.contactNum,p.heriditaryDiseasesIfAny,p.heriditaryDiseases,p.seriousAilmentIfAny,p.seriousAilment,p.otherDieseasesIfAny,p.otherDieseases,p.anySurgeryDone,p.SurgeryDone,p.masterCardPrint,p.invoiceRaised,p.collection,p.businessAreaId,p.localarea,p.district,SG.gender,MS.maritalStatus,SR.religion,BS.businessUnitName,
count(1) over() COUNTER
FROM 
pc_patient_registration p
LEFT JOIN sys_state s ON s.stateId=p.stateId
INNER JOIN sys_gender as SG on SG.genderId=p.genderId
LEFT OUTER JOIN sys_marital_status as MS on MS.maritalStatusId=p.maritalStatusId
LEFT OUTER JOIN sys_religion as SR on SR.religionId=p.religionId
INNER JOIN admin_business_area as BS on BS.businessAreaId=p.businessAreaId
 WHERE (DATE(p.createdOn)>=DATE(p_dateOfRegdExistingSystem) OR p_dateOfRegdExistingSystem IS NULL) AND (DATE(p.createdOn)<=DATE(p_dateOfRegd) OR p_dateOfRegd IS NULL)
-- AND p.businessAreaId=p_businessAreaId
AND (trim(p.uprnNo)=trim(p_uprnNo) OR p_uprnNo IS NULL)
AND (trim(p.AadharNo)=trim(p_AadharNo) OR p_AadharNo IS NULL)
AND (trim(p.mobileNo)=trim(p_mobileNo) OR p_mobileNo IS NULL)
AND (p.createdBy=p_createdBy OR p_createdBy IS NULL)
AND (p.stateId=p_stateId OR p_stateId =0 OR p_stateId IS NULL)
AND (TRIM(REPLACE(CONCAT(p.firstName,CASE WHEN p.middleName <> '' THEN CONCAT(' ', p.middleName) ELSE '' END,' ',p.lastName),'  ',' ')) LIKE CONCAT('%', TRIM(p_firstName), '%')  OR p_firstName IS NULL)
ORDER BY p.patientRegistrationId DESC limit p_pageno,p_perpage;
END IF;
END IF;
*/


 /*IF P_ACTIONFLAG = 'SEARCHCOUNTER' THEN
SELECT count(p.patientRegistrationId) as COUNTER
FROM 
pc_patient_registration p
LEFT JOIN sys_state s ON s.stateId=p.stateId
INNER JOIN sys_gender as SG on SG.genderId=p.genderId
WHERE (DATE(p.createdOn)>=DATE(p_dateOfRegdExistingSystem) OR p_dateOfRegdExistingSystem IS NULL) AND (DATE(p.createdOn)<=DATE(p_dateOfRegd) OR p_dateOfRegd IS NULL)
-- AND p.businessAreaId=p_businessAreaId
AND (p_uprnNo IS NULL OR p_uprnNo = '' OR TRIM(p.uprnNo) LIKE CONCAT('%', TRIM(p_uprnNo), '%'))
AND (trim(p.AadharNo)=trim(p_AadharNo) OR p_AadharNo IS NULL)
AND (trim(p.mobileNo)=trim(p_mobileNo) OR p_mobileNo IS NULL)
AND (p.createdBy=p_createdBy OR p_createdBy IS NULL)
AND (p.stateId=p_stateId OR p_stateId IS NULL OR p_stateId=0)
AND (TRIM(REPLACE(CONCAT(p.firstName,CASE WHEN p.middleName <> '' THEN CONCAT(' ', p.middleName) ELSE '' END,' ',p.lastName),'  ',' ')) LIKE CONCAT('%', TRIM(p_firstName), '%')  OR p_firstName IS NULL)
AND (p.uhidStatus=p_uhidStatus OR p_uhidStatus IS NULL)
AND (
    p_gender IS NULL
    OR
    CASE p_gender
        WHEN 'Male'         THEN SG.gender = 'Male'
        WHEN 'Female'       THEN SG.gender = 'Female'
        WHEN 'Transgenders' THEN SG.gender = 'Transgenders'
        ELSE 1 = 0
    END
)
AND (
    p_age IS NULL
    OR
    CASE p_age
        WHEN 1 THEN TIMESTAMPDIFF(YEAR, p.dateOfBirth, CURDATE()) BETWEEN 0 AND 5
        WHEN 2 THEN TIMESTAMPDIFF(YEAR, p.dateOfBirth, CURDATE()) BETWEEN 6 AND 10
        WHEN 3 THEN TIMESTAMPDIFF(YEAR, p.dateOfBirth, CURDATE()) BETWEEN 11 AND 15
        WHEN 4 THEN TIMESTAMPDIFF(YEAR, p.dateOfBirth, CURDATE()) BETWEEN 16 AND 20
        WHEN 5 THEN TIMESTAMPDIFF(YEAR, p.dateOfBirth, CURDATE()) BETWEEN 21 AND 25
        WHEN 6 THEN TIMESTAMPDIFF(YEAR, p.dateOfBirth, CURDATE()) BETWEEN 26 AND 30
        WHEN 7 THEN TIMESTAMPDIFF(YEAR, p.dateOfBirth, CURDATE()) BETWEEN 31 AND 35
        WHEN 8 THEN TIMESTAMPDIFF(YEAR, p.dateOfBirth, CURDATE()) BETWEEN 36 AND 40
        WHEN 9 THEN TIMESTAMPDIFF(YEAR, p.dateOfBirth, CURDATE()) BETWEEN 41 AND 45
        WHEN 10 THEN TIMESTAMPDIFF(YEAR, p.dateOfBirth, CURDATE()) BETWEEN 46 AND 50
        WHEN 11 THEN TIMESTAMPDIFF(YEAR, p.dateOfBirth, CURDATE()) BETWEEN 51 AND 55
        WHEN 12 THEN TIMESTAMPDIFF(YEAR, p.dateOfBirth, CURDATE()) BETWEEN 56 AND 60
        WHEN 13 THEN TIMESTAMPDIFF(YEAR, p.dateOfBirth, CURDATE()) BETWEEN 61 AND 65
        WHEN 14 THEN TIMESTAMPDIFF(YEAR, p.dateOfBirth, CURDATE()) BETWEEN 66 AND 70
        WHEN 15 THEN TIMESTAMPDIFF(YEAR, p.dateOfBirth, CURDATE()) BETWEEN 71 AND 75
        WHEN 16 THEN TIMESTAMPDIFF(YEAR, p.dateOfBirth, CURDATE()) > 75
        ELSE 1 = 0
    END
);
END IF; */

IF P_ACTIONFLAG = 'SEARCHCOUNTER' THEN

SELECT COUNT(DISTINCT p.patientRegistrationId) AS COUNTER
FROM 
pc_patient_registration p
LEFT JOIN sys_state s ON s.stateId = p.stateId
INNER JOIN sys_gender AS SG ON SG.genderId = p.genderId

WHERE 
(DATE(p.createdOn) >= DATE(p_dateOfRegdExistingSystem) OR p_dateOfRegdExistingSystem IS NULL)
AND (DATE(p.createdOn) <= DATE(p_dateOfRegd) OR p_dateOfRegd IS NULL)

AND (p.businessAreaId = p_businessAreaId OR p_businessAreaId IS NULL OR p_businessAreaId = 0)

AND (
    p_uprnNo IS NULL OR TRIM(p_uprnNo) = ''
    OR TRIM(p.uprnNo) LIKE CONCAT('%', TRIM(p_uprnNo), '%')
)

AND (
    p_AadharNo IS NULL OR TRIM(p_AadharNo) = ''
    OR TRIM(p.AadharNo) = TRIM(p_AadharNo)
)

AND (
    p_mobileNo IS NULL OR TRIM(p_mobileNo) = ''
    OR TRIM(p.mobileNo) = TRIM(p_mobileNo)
)

AND (p.createdBy = p_createdBy OR p_createdBy IS NULL OR p_createdBy = 0)

AND (p.stateId = p_stateId OR p_stateId IS NULL OR p_stateId = 0)

AND (
    p_firstName IS NULL OR TRIM(p_firstName) = ''
    OR TRIM(REPLACE(CONCAT(
        p.firstName,
        CASE WHEN p.middleName <> '' THEN CONCAT(' ', p.middleName) ELSE '' END,
        ' ', p.lastName
    ), '  ', ' ')) LIKE CONCAT('%', TRIM(p_firstName), '%')
)

AND (p.uhidStatus = p_uhidStatus OR p_uhidStatus IS NULL)

AND (
    p_gender IS NULL OR TRIM(p_gender) = ''
    OR
    CASE p_gender
        WHEN 'Male' THEN SG.gender = 'Male'
        WHEN 'Female' THEN SG.gender = 'Female'
        WHEN 'Transgenders' THEN SG.gender = 'Transgenders'
        ELSE 1 = 0
    END
)

AND (
    p_age IS NULL OR p_age = 0
    OR
    CASE p_age
        WHEN 1 THEN TIMESTAMPDIFF(YEAR, p.dateOfBirth, CURDATE()) BETWEEN 0 AND 5
        WHEN 2 THEN TIMESTAMPDIFF(YEAR, p.dateOfBirth, CURDATE()) BETWEEN 6 AND 10
        WHEN 3 THEN TIMESTAMPDIFF(YEAR, p.dateOfBirth, CURDATE()) BETWEEN 11 AND 15
        WHEN 4 THEN TIMESTAMPDIFF(YEAR, p.dateOfBirth, CURDATE()) BETWEEN 16 AND 20
        WHEN 5 THEN TIMESTAMPDIFF(YEAR, p.dateOfBirth, CURDATE()) BETWEEN 21 AND 25
        WHEN 6 THEN TIMESTAMPDIFF(YEAR, p.dateOfBirth, CURDATE()) BETWEEN 26 AND 30
        WHEN 7 THEN TIMESTAMPDIFF(YEAR, p.dateOfBirth, CURDATE()) BETWEEN 31 AND 35
        WHEN 8 THEN TIMESTAMPDIFF(YEAR, p.dateOfBirth, CURDATE()) BETWEEN 36 AND 40
        WHEN 9 THEN TIMESTAMPDIFF(YEAR, p.dateOfBirth, CURDATE()) BETWEEN 41 AND 45
        WHEN 10 THEN TIMESTAMPDIFF(YEAR, p.dateOfBirth, CURDATE()) BETWEEN 46 AND 50
        WHEN 11 THEN TIMESTAMPDIFF(YEAR, p.dateOfBirth, CURDATE()) BETWEEN 51 AND 55
        WHEN 12 THEN TIMESTAMPDIFF(YEAR, p.dateOfBirth, CURDATE()) BETWEEN 56 AND 60
        WHEN 13 THEN TIMESTAMPDIFF(YEAR, p.dateOfBirth, CURDATE()) BETWEEN 61 AND 65
        WHEN 14 THEN TIMESTAMPDIFF(YEAR, p.dateOfBirth, CURDATE()) BETWEEN 66 AND 70
        WHEN 15 THEN TIMESTAMPDIFF(YEAR, p.dateOfBirth, CURDATE()) BETWEEN 71 AND 75
        WHEN 16 THEN TIMESTAMPDIFF(YEAR, p.dateOfBirth, CURDATE()) > 75
        ELSE 1 = 0
    END
);

END IF;

IF P_ACTIONFLAG = 'UPLOAD_DEATHCERTIFICATE' THEN       
 UPDATE pc_patient_registration SET deathStatus='Y',deathCertificate=p_deathCertificate,deathDate=p_deathDate,deathRemark=p_deathRemark where uprnNo=p_uprnNo AND status=0;  
 SELECT ROW_COUNT() INTO lastInsertId;
 IF lastInsertId  >= 1 THEN
 SELECT "Updated Successfully" into p_yes_no;
 else 
 SELECT "Value not updated" into p_yes_no;
END IF;
END IF;

IF P_ACTIONFLAG = 'VIEWDEATHDETAILS' THEN 
SELECT PR.patientRegistrationId,PR.uprnNo,PR.dateOfRegd,concat(PR.firstName,' ',PR.middleName,' ',PR.lastName) PatientName,PR.firstName,PR.middleName,PP.drEmployeeId,PR.lastName,PR.mobileNo,PR.mobileNoValidate,PR.patientPhoto,SS.stateName,SG.gender,MS.maritalStatus,SR.religion,BS.businessUnitName,PR.status,PR.localarea,PR.district,PR.fathersName,PR.deathStatus,PR.deathDate,PR.deathRemark,PR.AadharNo
FROM pc_patient_registration PR
INNER JOIN pc_patient_ticket_registration_empn PP on  PP.patientRegistrationId = PR.patientRegistrationId
INNER JOIN sys_state SS on SS.stateId=PR.stateId
INNER JOIN sys_gender SG on SG.genderId=PR.genderId
LEFT OUTER JOIN sys_marital_status MS on MS.maritalStatusId=PR.maritalStatusId
LEFT OUTER JOIN sys_religion SR on SR.religionId=PR.religionId	
INNER JOIN admin_business_area BS on BS.businessAreaId=PR.businessAreaId
WHERE  PR.status=0 AND PR.deathStatus='Y' order by  PR.patientRegistrationId;
END IF;
 
IF P_ACTIONFLAG = 'DUPLICATECOUNT' THEN
IF(SELECT COUNT(patientRegistrationId) FROM pc_patient_registration WHERE TRIM(UPPER(firstName))=TRIM(UPPER(p_firstName)) AND TRIM(UPPER(lastName))=TRIM(UPPER(p_lastName)) AND TRIM(UPPER(fathersName))=TRIM(UPPER(p_fathersName)) and TRIM(mobileNo)=TRIM(p_mobileNo) AND status=0)>=1 THEN 
SELECT "Patient Registration Already Exist" INTO p_yes_no; 
END IF;
END IF;
    
IF P_ACTIONFLAG = 'EXISTINGSYSTEMDATE' THEN 
SELECT patientRegistrationId,dateOfRegdExistingSystem,createdOn FROM pc_patient_registration ORDER BY patientRegistrationId ASC LIMIT 1; 
END IF;

/*

select `Doctor Name`, sum(`Cons. Fees(OPD)`)`Cons. Fees(OPD)`, sum(`Cons. Fees(IPD)`)`Cons. Fees(IPD)`, sum(`Total Cons. Fees`)`Total Cons. Fees` from 
(
SELECT TRIM(docName)`Doctor Name`, `consFees(OPD)` `Cons. Fees(OPD)`, `consFees(IPD)` `Cons. Fees(IPD)`,(`consFees(OPD)`+`consFees(IPD)`)`Total Cons. Fees`
FROM
(select e.fullName docName,coalesce(sum(case when i.serviceCatagoryId=1 and a.type='O' then b.total end),0)`consFees(OPD)`,coalesce(sum(case when i.serviceCatagoryId=1 and a.type='I' then b.total end),0)`consFees(IPD)`
from 
pc_cash_billing a 
inner join pc_cash_billing_details b on b.cashBillingId=a.cashBillingId 
inner join admin_service_item f on f.serviceItemId=b.serviceItemId 
inner join admin_service_master g on g.serviceId=f.serviceId 
inner join admin_service_sub_catagory h on h.serviceSubCatagoryId=g.serviceSubCatagoryId 
inner join admin_service_catagory i on i.serviceCatagoryId=h.serviceCatagoryId
inner join admin_service_classification j on j.serviceClassificationId=i.serviceClassificationId 
left join hrms_employee_registration e on e.userId=a.treatDrEmployeeId
where a.businessAreaId=1 and date(a.createdOn) between '2026-10-01' and '2026-10-31' and a.status=0 and a.pendingStatus <> 'C' 
group by e.fullName
)T WHERE docName IS NOT NULL

union all 
SELECT TRIM(docName)`Doctor Name`, `consFees(OPD)` `Cons. Fees(OPD)`, `consFees(IPD)` `Cons. Fees(IPD)`,(`consFees(OPD)`+`consFees(IPD)`)`Total Cons. Fees`
FROM
(select e.fullName docName,coalesce(sum(case when i.serviceCatagoryId=1 and a.patientType='O' then b.total end),0)`consFees(OPD)`,coalesce(sum(case when i.serviceCatagoryId=1 and a.patientType='I' then b.total end),0)`consFees(IPD)`
from 
pc_bsky_internal_billing a 
inner join pc_bsky_internal_billing_details b on b.internalBillingId=a.internalBillingId 
inner join admin_service_item f on f.serviceItemId=b.serviceItemId 
inner join admin_service_master g on g.serviceId=f.serviceId 
inner join admin_service_sub_catagory h on h.serviceSubCatagoryId=g.serviceSubCatagoryId 
inner join admin_service_catagory i on i.serviceCatagoryId=h.serviceCatagoryId
inner join admin_service_classification j on j.serviceClassificationId=i.serviceClassificationId 
left join hrms_employee_registration e on e.userId=a.treatDrEmployeeId
where a.businessAreaId=1 and date(a.createdOn) between '2026-10-01' and '2026-10-31' and a.status=0 and a.pendingStatus <> 'C' 
group by e.fullName
)T WHERE docName IS NOT NULL
union all 
SELECT TRIM(docName)`Doctor Name`, `consFees(OPD)` `Cons. Fees(OPD)`, `consFees(IPD)` `Cons. Fees(IPD)`,(`consFees(OPD)`+`consFees(IPD)`)`Total Cons. Fees`
FROM
(select e.fullName docName,coalesce(sum(case when i.serviceCatagoryId=1 and a.patientType='O' then b.total end),0)`consFees(OPD)`,coalesce(sum(case when i.serviceCatagoryId=1 and a.patientType='I' then b.total end),0)`consFees(IPD)`
from 
pc_corporate_cash_billing a 
inner join pc_corporate_cash_billing_details b on b.corporateCashBillingId=a.corporateCashBillingId 
inner join admin_service_item f on f.serviceItemId=b.serviceItemId 
inner join admin_service_master g on g.serviceId=f.serviceId 
inner join admin_service_sub_catagory h on h.serviceSubCatagoryId=g.serviceSubCatagoryId 
inner join admin_service_catagory i on i.serviceCatagoryId=h.serviceCatagoryId
inner join admin_service_classification j on j.serviceClassificationId=i.serviceClassificationId 
left join hrms_employee_registration e on e.userId=a.treatDrEmployeeId
where a.businessAreaId=1 and date(a.createdOn) between '2026-10-01' and '2026-10-31' and a.status=0 and a.pendingStatus <> 'C' 
group by e.fullName
)T WHERE docName IS NOT NULL
union all 
SELECT TRIM(docName)`Doctor Name`, `consFees(OPD)` `Cons. Fees(OPD)`, `consFees(IPD)` `Cons. Fees(IPD)`,(`consFees(OPD)`+`consFees(IPD)`)`Total Cons. Fees`
FROM
(select e.fullName docName,coalesce(sum(case when i.serviceCatagoryId=1 and a.patientType='O' then b.total end),0)`consFees(OPD)`,coalesce(sum(case when i.serviceCatagoryId=1 and a.patientType='I' then b.total end),0)`consFees(IPD)`
from 
pc_insurance_internal_billing a 
inner join pc_insurance_internal_billing_details b on b.insuranceInternalBillingId=a.insuranceInternalBillingId 
inner join admin_service_item f on f.serviceItemId=b.serviceItemId 
inner join admin_service_master g on g.serviceId=f.serviceId 
inner join admin_service_sub_catagory h on h.serviceSubCatagoryId=g.serviceSubCatagoryId 
inner join admin_service_catagory i on i.serviceCatagoryId=h.serviceCatagoryId
inner join admin_service_classification j on j.serviceClassificationId=i.serviceClassificationId 
left join hrms_employee_registration e on e.userId=a.treatDrEmployeeId
where a.businessAreaId=1 and date(a.createdOn) between '2026-10-01' and '2026-10-31' and a.status=0 and a.pendingStatus <> 'C' 
group by e.fullName
)T WHERE docName IS NOT NULL
)t group by `Doctor Name` order by `Doctor Name`;

*/
END
