## [HQ-RTR]

### Привязка VLAN-интерфейсов к физическим интерфейсам
vl120
port ge1
 service-instance ge1/v120
  encapsulation dot1q 120 exact
  rewrite pop 1
  connect ip interface vlan120
 exit
vl220
 service-instance ge1/v220
  encapsulation dot1q 220 exact
  rewrite pop 1
  connect ip interface vlan220
 exit
vl888
 service-instance ge1/v888
  encapsulation dot1q 888 exact
  rewrite pop 1
  connect ip interface vlan888
 exit
exit

