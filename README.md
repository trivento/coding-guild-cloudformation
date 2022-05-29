# coding-guild-cloudformation
cloudformation demo and presentation for the coding guild of Trivento given on 30 mei 2022
Deze repo is terug te vinden op: https://github.com/trivento/coding-guild-cloudformation

De keynote presentatie is terug te vinden in het bestand: "Coding Guild - IaC Infrastructure as Code.key"

# voorbereiding
Er is geen voorbereiding nodig, want we doen alles vanuit de AWS UI. 
Een user binnen AWS is echter wel aan te bevelen. 

## AWS CLI
Het gebruik van de AWS CLI is wel prettig. 
Hoe je dit opzet is te vinden in: https://github.com/trivento/cloudformation
tip: maak je credentials voor de CLI voor deze codingguild aan onder de naam `codingguild`, dan kan je de commando's kopieren en plakken, zonder problemen.

`~/.aws/config`
```
[profile codingguild]
region = eu-central-1
output = json
```

`~/.aws/credentials`
```
#Trivento coding guild account
[codingguild]
aws_access_key_id = *** masked ***
aws_secret_access_key = *** masked ***
```

Belangrijk is om bij elk AWS CLI commando een --profile mee te geven

```
# create a stack
aws cloudformation --profile codingguild create-stack --stack-name myteststack --template-body file://sampletemplate.yaml

# List all stacks
aws cloudformation --profile codingguild list-stacks --query "StackSummaries[*].StackName"

# Delete a stack by name
aws cloudformation --profile codingguild delete-stack --stack-name test-sebas
```

## linter
Installeer een linter in VScode
```
pip install cfn-lint

# en optioneel om templates als graphs te kunnen zien
pip install pydot
```
https://github.com/aws-cloudformation/cfn-lint-visual-studio-code


# demo stack
Om te voorkomen dat iedereen een stack aanmaakt met de zelfde naam, wat overigens errors gaat geven.
Doe even een search and replace op `myteststack` en verzin even een eigen stack naam. Dan kom je niet voor verrassingen te staan met kopieren en plakken van commando's

Een call naar je API maken.
1. zoek je url op in de outputs van je stack, dit is de key: `apiGatewayInvokeURL`
   vb: `https://8oij7jc8nl.execute-api.eu-central-1.amazonaws.com/call`
2. maak een curl call naar deze url
   vb: `curl --request POST https://8oij7jc8nl.execute-api.eu-central-1.amazonaws.com/call`

   De response is dan: `Hello there <your ip>`

## of met de CLI
De optie --capabilities CAPABILITY_NAMED_IAM is voor deze stack nodig omdat er IAM roles en policies aangemaakt worden voor de Lambda functie. 

```
# create the stack
aws cloudformation --profile codingguild deploy --template-file 01-Simple-API.yml --stack-name myteststack --capabilities CAPABILITY_NAMED_IAM

# get the output value for the API URL
# change the StackName to your stack name..
aws cloudformation --profile codingguild describe-stacks --query 'Stacks[?StackName==`myteststack`][].Outputs[?OutputKey==`apiGatewayInvokeURL`].OutputValue' --output text

# het curl commando
curl --request POST $(aws cloudformation --profile codingguild describe-stacks --query 'Stacks[?StackName==`myteststack`][].Outputs[?OutputKey==`apiGatewayInvokeURL`].OutputValue' --output text)

# delete your stack
aws cloudformation --profile codingguild delete-stack --stack-name myteststack
```

# demo2 stack
Een cloudformation stack waarbij de parameters en tags meegegeven worden als losse bestanden aan het `aws cloudformation deploy` commando. 
Het mooie is zo dat je de waarden van de parameters op deze manier dus ook mooi in code en in GIT kan vastleggen
```
cd '02 more complex'
```

Create the stack
`deploy` kan een stack zowel creeren als updaten. 
```
aws cloudformation --profile codingguild deploy --template-file 02-templated-stack.yml --stack-name myteststack --capabilities CAPABILITY_NAMED_IAM --parameter-overrides file://parameters.json --tags $(cat tags.ini)

#even voor mijn eigen test
aws cloudformation --profile codingguild deploy --template-file 02-templated-stack.yml --stack-name nogeenstack --capabilities CAPABILITY_NAMED_IAM --parameter-overrides file://parameters.json --tags $(cat tags.ini)
curl --request POST $(aws cloudformation --profile codingguild describe-stacks --query 'Stacks[?StackName==`nogeenstack`][].Outputs[?OutputKey==`apiGatewayInvokeURL`].OutputValue' --output text)
```

Haal nu alle resources op met een bepaalde tag
Dit werkt helaas niet goed. 
Er komt een error:
`when calling the GetResources operation: User: arn:aws:iam::961488927380:user/codingguild-sebas is not authorized to perform: tag:GetResources because no identity-based policy allows the tag:GetResources action`
Maar het toevoegen van de action `tag:GetResources` lost het probleem niet op. Het toevoegen van full administrator rechten werkt wel. Het probleem is dus niet, wat de error weergeeft. 
```
aws resourcegroupstaggingapi --profile codingguild get-resources --tag-filters filter1={employee{sebastiaan}}
```

# resources
https://bl.ocks.org/magnetikonline/c314952045eee8e8375b82bc7ec68e88
https://www.1strategy.com/blog/2019/02/20/cloudformation-ition-made-simple/