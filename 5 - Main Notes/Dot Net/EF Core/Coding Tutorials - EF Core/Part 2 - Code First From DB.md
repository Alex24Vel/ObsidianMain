Source: https://youtu.be/-UdY-dcBv6M?list=PLQB-TSatJvw4T7mQneItRgsemyjMMYRNk

Date: 26.04.2025 13:29

## Notes

Scaffolding the entity models and then adding business logic to them is wrong.

Create some folder, for instance 'Additions' and declare there the same partial classes entities that got scaffolded. In those 'Addition' classes you add your business logic.

Why the overhead? - whenever the database gets changed - scaffolded entities may change - business logic may be lost. With business logic defined elsewhere we don't have to fear loosing the existing code.