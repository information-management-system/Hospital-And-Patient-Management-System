User story 1:

To maintain Doctor’s information

db.doctorInformation.insert(
{
   "doctorId" : "d3",
   "name" : "Dr. Maria Thomas",
   "qualification" : "MD - Skin,VD & Leprosy, MBBS",
   "experience" : "11years",
   "specialization" : "Dermatology",
   "publications" : [ 
       "Israel Medical Association Journal", 
       "JAMA Dermatology"
   ],
   "awards" : [ 
       "Dr. Sardari Lal Memorial Award", 
       "Aronson Prize"
   ],
       "researchPaper" : [ 
       "American Journal of Translational Research", 
       "Research and Humanities in Medical Education"
   ],
   "start" : "2018-05-05T13:00:00",
   "timeZone" : "Europe/Germany",
   "doctorTimeSlots" : {
       "2019-03-01T13:00:00" : 0.0,
       "2019-02-01T13:00:00" : 1.0,
       "2019-06-25T13:00:00" : 0.0,
       "2019-08-12T13:00:00" : 1.0,
       "2019-10-19T13:00:00" : 0.0,
       "2019-07-05T13:00:00" : 1.0
   }
}
)

OUTPUT -
WriteResult({ "nInserted" : 1 })

To fetch doctor’s availability on 12-08-2019 with doctor Id, name and specialization. 

QUERY -

db.doctorInformation.find({"doctorTimeSlots.2019-08-12T13:00:00" : 1},{"doctorId" : 1, "name" : 1,"specialization" : 1}).pretty()

OUTPUT –
{
	"_id" : ObjectId("5ccaa52d1547188f81a36f1d"),
	"doctorId" : "d1",
	"name" : "Dr. Ramesh Reddy",
	"specialization" : "Surgery"
}
{
	"_id" : ObjectId("5ccc157d787e925ca53a341b"),
	"doctorId" : "d3",
	"name" : "Dr. Maria Thomas",
	"specialization" : "Dermatology"
}

To fetch which doctor is treating which patient.

QUERY -

db.patientInformation.aggregate([
   { $lookup:
      {
        from: 'doctorInformation',
        localField: 'doctorId',
        foreignField: 'doctorId',
        as: 'join'
      }
    },
    {$unwind:'$join'},
    {
       "$project": {
         "doctorId": 1,
         "firstName": 1,
         "lastName":1,
         "occupation": 1,
         "doctorName": '$join.name',
         "doctorSpecialization":'$join.specialization'
       }
     }
])

OUTPUT –
{
	"_id" : ObjectId("5cc198850e56351dadc25dc6"),
	"doctorId" : "d1",
	"firstName" : "Paul",
	"lastName" : "Walter",
	"occupation" : "civilEngineer",
	"doctorName" : "Dr. Ramesh Reddy",
	"doctorSpecialization" : "Surgery"
}
{
	"_id" : ObjectId("5cc199340e56351dadc25dc7"),
	"doctorId" : "d2",
	"firstName" : "John",
	"lastName" : "Gerhard",
	"occupation" : "painter",
	"doctorName" : "Dr. Alexander Graf",
	"doctorSpecialization" : "Neurology"
}
{
	"_id" : ObjectId("5cc6df510847954df01037e6"),
	"doctorId" : "d1",
	"firstName" : "Paul",
	"lastName" : "Walter",
	"occupation" : "civilEngineer",
	"doctorName" : "Dr. Ramesh Reddy",
	"doctorSpecialization" : "Surgery"
}
{
	"_id" : ObjectId("5cc6fa1d0847954df01037e8"),
	"doctorId" : "d1",
	"firstName" : "Emma",
	"lastName" : "Lina",
	"occupation" : "electricalEngineer",
	"doctorName" : "Dr. Ramesh Reddy",
	"doctorSpecialization" : "Surgery"
}


To fetch the emergency case of type accident.

QUERY -

db.patientInformation.aggregate([
{$match: {"emergencyCase.typeOfIncident": "accident" }},
{$group: {_id: { FirstName: "$firstName", 
   LastName: "$lastName",
   Age: "$age",
   Gender: "$gender",
   Occupation: "$occupation",
   emergency: "$emergencyCase"
    }} }
]).pretty()

OUTPUT –
{
	"_id" : {
		"FirstName" : "Sam",
		"LastName" : "Wilson",
		"Age" : "27",
		"Gender" : "male",
		"Occupation" : "civilEngineer",
		"emergency" : {
			"typeOfIncident" : "accident"
		}
	}
}

User story 2:

a. To fetch the patient’s current disease with their lab reports

QUERY –

 db.patientInformation.aggregate([
    {$match: {"firstName" : "Paul"}},
    {$group: {_id: {patientInformation : { firstName: "$firstName", 
        LastName: "$lastName",
        Age: "$age",
        Gender: "$gender",
        Occupation: "$occupation",
        currentDisease : "$currentDisease",
        currentLabReports : "$currentLabReports"
        }} } }
    ]).pretty()
 
 
OUTPUT –
{
	"_id" : {
		"patientInformation" : {
			"firstName" : "Paul",
			"LastName" : "Walter",
			"Age" : "27",
			"Gender" : "male",
			"Occupation" : "civilEngineer",
			"currentDisease" : {
				"disease" : "malaria-chronic",
				"typeOfDisease" : "plasmodium malariae",
				"symptoms" : "fever, vomiting",
				"dietPlanSuggested" : "eat carrot, beetroots, papaya, berries",
				"medicinesPrescribed" : "mefloquine, quinine"
			},
			"currentLabReports" : {
				"bloodTest" : {
					"haemoglobin" : "positive, 14.5 gm/dl",
					"malaria antigen blood test" : "positive, non falciparum plasmodium species",
					"bilirubin" : "negative, 6/4.7 mg/dl"
				},
				"thyroidTest" : {
					"throglobin" : "positive, 5 IU/ml",
					"throxine" : "negative, 7.0 DNR mg/dl",
					"tsh" : "negative, 1.5 mIU/ml"
				},
				"diagnosticTest" : {
					"latex culture" : "negative, 66(10.3)%",
					"rt PCR" : "negative, 74(94.0)%"
				}
			}
		}
	}
}
 
 
b. To fetch the number of people with same disease type.
 
QUERY –
 
db.patientInformation.aggregate([
    {$match: {"currentDisease.disease" : "malaria-chronic","medicalHistory.disease": "malaria" }},
    {$group: {_id: { firstName: "$firstName", 
        LastName: "$lastName",
        Age: "$age",
        Gender: "$gender",
        Occupation: "$occupation",
                        pastDisease:  "$medicalHistory",
                        currentDisease: "$currentDisease",
        currentLabReports : "$currentLabReports"
       
        }} }
    ]
    ).pretty()
 
 
OUTPUT –
{ 
    "_id" : {
        "firstName" : "John", 
        "LastName" : "Gerhard", 
        "Age" : "32", 
        "Gender" : "male", 
        "Occupation" : "painter", 
        "pastDisease" : {
            "_id" : ObjectId("5ccae5fb12a1fe1754218155"), 
            "disease" : "malaria", 
            "medicinesPrescribed" : "acetaminophen", 
            "dietPlan" : "drink coconut water, avoid spicy foods", 
            "dateWhenDiseaseOccurred" : "27.02.2019", 
            "labReports" : "bloodTest"
        }, 
        "currentDisease" : {
            "disease" : "malaria-chronic", 
            "typeOfDisease" : "typhoid-O", 
            "symptoms" : "abdominal pain, severe headache", 
            "dietPlan" : "eat fruits, glucose water, soft rice and more water", 
            "medicinesPrescribed" : "ceftriaxone, ciprofloxacin"
        }, 
        "currentLabReports" : {
            "bloodCultureTest" : {
                "_id" : ObjectId("5ccae55412a1fe1754218150"), 
                "test group blood culture" : "positive 104 Igm", 
                "control group III" : "negative 22 IgM"
            }, 
            "haemotologicalTest" : {
                "_id" : ObjectId("5ccae66912a1fe1754218165"), 
                "WBC Count" : "postive, 8(10.6) /ul", 
                "haemoglobin" : "negative, 46(61.3) gm/dl", 
                "platelet count" : "negative, 45(60) /ul"
            }
        }
    }
}
// ----------------------------------------------
{ 
    "_id" : {
        "firstName" : "Paul", 
        "LastName" : "Walter", 
        "Age" : "27", 
        "Gender" : "male", 
        "Occupation" : "civilEngineer", 
        "pastDisease" : {
            "_id" : ObjectId("5ccae63b12a1fe175421815c"), 
            "disease" : "malaria", 
            "medicinesPrescribed" : "malarone", 
            "dietPlan" : "eat fruits, avoid junk and processed foods", 
            "dateWhenDiseaseOccurred" : "20.02.2019", 
            "labReports" : "PCR"
        }, 
        "currentDisease" : {
            "disease" : "malaria-chronic", 
            "typeOfDisease" : "plasmodium malariae", 
            "symptoms" : "fever, vomiting", 
            "dietPlan" : "eat carrot, beetroots, papaya, berries", 
            "medicinesPrescribed" : "mefloquine, quinine"
        }, 
        "currentLabReports" : {
            "bloodTest" : {
                "_id" : ObjectId("5ccae5fb12a1fe1754218155"), 
                "haemoglobin" : "positive, 14.5 gm/dl", 
                "malaria antigen blood test" : "positive, non falciparum plasmodium species", 
                "bilirubin" : "negative, 6/4.7 mg/dl"
            }, 
            "thyroidTest" : {
                "_id" : ObjectId("5ccae65a12a1fe1754218162"), 
                "throglobin" : "positive, 5 IU/ml", 
                "throxine" : "negative, 7.0 DNR mg/dl", 
                "tsh" : "negative, 1.5 mIU/ml"
            }
        }
    }
}
 
 
User story 3:

Product insert query:
hmset product1 name "cottonRoll" manufacturing_date 20.03.2019 expiry_date 27.03.2019 quantity 2000 manufacturer "Apollo"

Pushing products into list:
lpush productsInformation product1
lpush productsInformation product2
lpush productsInformation product3
lpush productsInformation product4
lpush productsInformation product5
lpush productsInformation product6


OUTPUT –

![r1](https://user-images.githubusercontent.com/45422560/57444415-65a87600-7250-11e9-8f6c-a69dc1dcb579.png)

![r2](https://user-images.githubusercontent.com/45422560/57444569-bc15b480-7250-11e9-9443-07d4abe957a4.png)


User story 4:

To fetch ward availability

QUERY –

MATCH (n:ward),(o:floor),(p:hospital) 
WHERE('available' IN labels(n))
RETURN *

OUTPUT -

![n1](https://user-images.githubusercontent.com/45422560/57444751-229ad280-7251-11e9-97b4-84bdd7d6330f.png)

To allocate bed to a patient

QUERY –

Match (n:ward) where n.wardNo="W7" Remove n:available set n.bed22="new ptn 2" set n:full 
return (n)

OUTPUT -

![n2](https://user-images.githubusercontent.com/45422560/57444782-380ffc80-7251-11e9-88d3-44bb0a391277.png)

To fetch number beds in the ward.

QUERY –

Match (n:floor)-[r:contains]->(m:ward) Where n.number = "Floor 3" Return m.wardNo,r.noOfbeds As NO_OF_BEDS

OUTPUT -

![n3](https://user-images.githubusercontent.com/45422560/57444802-452ceb80-7251-11e9-8a2f-9f59305af3d9.png)

User story 5:

To fetch staff information with their staff Id, name and work type

QUERY -

db.staffInformation.aggregate([
{$project: {staffId : 1, firstName : 1, workType : 1, _id:0, }}
])


OUTPUT –
{ 
    "staffId" : "s1", 
    "firstName" : "Harry", 
    "workType" : "technical"
}
// ----------------------------------------------
{ 
    "staffId" : "s1", 
    "firstName" : "Jacob", 
    "workType" : "technical"
}
// ----------------------------------------------
{ 
    "staffId" : "s1", 
    "firstName" : "Mila", 
    "workType" : "technical"
}
// ----------------------------------------------
{ 
    "staffId" : "s2", 
    "firstName" : "Jack", 
    "workType" : "nonTechnical"
}
// ----------------------------------------------
{ 
    "staffId" : "s2", 
    "firstName" : "Moritz", 
    "workType" : "nonTechnical"
}
// ----------------------------------------------
{ 
    "staffId" : "s2", 
    "firstName" : "Henry", 
    "workType" : "nonTechnical"
}

To fetch the count of both technical and non-technical staff members

QUERY –

db.staffInformation.aggregate([
{$match: {workType:"technical"}},
{$group: {
    	_id:"$staffId",
    	numberOfTechnicalStaff: {$sum : 1 }
		 }
}     
])

db.staffInformation.aggregate([
{$match: {workType:"nonTechnical"}},
{$group: {
    	_id:"$staffId",
    	numberOfNonTechnicalStaff: {$sum : 1 }
		 }
}     
])


OUTPUT –
{ 
    "_id" : "s1", 
    "numberOfTechnicalStaff" : 3.0
}

{ 
    "_id" : "s2", 
    "numberOfNonTechnicalStaff" : 3.0
}


To fetch the staff availability using calendar data structure for 2019-03-28T12:00:00
QUERY –

db.staffInformation.find({"staffAvailability.2019-03-28T12:00:00" : 1},{"staffId" : 1, "firstName" : 1, "workType" : 1, "_id" : 0})

OUTPUT –
{ 
    "staffId" : "s1", 
    "firstName" : "Harry", 
    "workType" : "technical"
}
// ----------------------------------------------
{ 
    "staffId" : "s1", 
    "firstName" : "Jacob", 
    "workType" : "technical"
}
// ----------------------------------------------
{ 
    "staffId" : "s2", 
    "firstName" : "Jack", 
    "workType" : "nonTechnical"
}

To fetch the staff information using their name

QUERY –

db.staffInformation.aggregate([
{$match: {firstName : "Harry"}},
{$project: {"staffId" : 1, "workType" : 1, "staffAvailability" : 1, "_id" : 0}}
])

OUTPUT –
{ 
    "staffId" : "s1", 
    "workType" : "technical", 
    "staffAvailability" : {
        "2019-02-01T14:00:00" : 0.0, 
        "2019-02-02T08:00:00" : 1.0, 
        "2019-03-01T10:00:00" : 1.0, 
        "2019-03-03T08:00:00" : 1.0, 
        "2019-03-28T12:00:00" : 1.0
    }
}
