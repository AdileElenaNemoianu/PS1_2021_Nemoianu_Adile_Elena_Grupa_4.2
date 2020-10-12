# PS1_2021_Nemoianu_Adile_Elena_Grupa_4.2
#Tema 2 de la laborator
#AAcesta este Codul care a fost deja simulat pe Tinkercad

#include <LiquidCrystal.h>
static LiquidCrystal lcd(12, 11, 5, 4, 3, 2); //initializarea LCD-ului cu valorile pinilor

int volatile temp1 = 0, temp2; //declaram variabilele pe care le vom folosi pentru studiul temperaturii
char ora[5]="12:", minut[5]="00:", secunda[5]="00"; // fixam ora 12

void Analog_to_Digital_Cnoverted_Initialized() //initializarea microcontrolerului folosind registrii 
{
	ADCSRA |= 1 << ADEN;   // activare ADC
  	ADMUX |= 1 << REFS0;   // folosim AVcc ca referinta
	ADCSRA |= 1 << ADIE;   // activarea intreruperilor 
  	ADCSRA |= 1 << ADSC;   // porneste conversia ADC = Analog to Digital Cnoverted

}

ISR(Analog_to_Digital_Cnoverted_vect)  //rutina pentru tratarea intreruperilor ... pentru temperatura
{
  temp1 = ADC;               // citim valoarea pe 8 biti
  float a = 5000.0/1023;    // adica a = 5 volti / 1023 unitati  ...cand se citesc 5 volti, microcontrolerul intelege 5000 mV... 
  temp2 = temp1*a;
  temp2 = temp2/10-50;      // 10 reprezinta unitatea la care temperatura se modifica cu un grad si -50 este valoarea minima pe care o poate masura senzorul LM35 de tensiune
  ADCSRA |= 1 << ADSC;     // incepe conversia ADC = Analog to Digital Cnoverte
}

ISR(TIMER1_COMPA_vect) // rutina pentru tratarea intreruperilor ... pentru timp(ora, minute, secunde)
{
  char memorie_temporara[20] = "Temperatura ", aux[4]; //
  itoa(temp2, aux, 10); // convertim valoarea int a temperaturii temp 2 in sirul aux (folosim baza de numeratie 10)
  strcat(memorie_temporara, aux); // salvam sirul aux in memoria temporara
  strcat(memorie_temporara, "'C"); // adaugam la sirul initial semnul Gradelor Celsius
  lcd.setCursor(0, 0); //setam cursorul 
  lcd.print(memorie_temporara); // afisam ceea ce am salvat in memoria temporara -- adica afisam temperatura 
  strcpy(memorie_temporara, "Ora ");  // copiem valoarea orei in memoria temporara
  strcat(memorie_temporara, ora);  // copiem valoarea orei in memoria temporara
  strcat(memorie_temporara, minut); // copiem valoarea minutului in memoria temporara
  strcat(memorie_temporara, secunda); // // copiem valoarea secundei in memoria temporara
  lcd.setCursor(0, 1); //setam cursorul
  lcd.print(memorie_temporara); //afisam ce am salvat in memoria temporara
  
  if(secunda[1] == '9')    // rutina necesara afisarii pe ecranul LCD a timpului format din ore, minute, secunde
  {
    if(secunda[0] == '5')
    {
      if(minut[1] == '9')
      {
        if(minut[0] == '5')
        {
          if(ora[1] == '3' && ora[0] == '2')
          {
            strcpy(ora, "00:");
          }
          else if(ora[1] == '9')	
          {
            ora[0]++;
            ora[1] = '0';
          }
          else
          {
            ora[1]++;
          }
          minut[0] = '0';
          minut[1] = '0';
          secunda[0] = '0';
          secunda[1] = '0';
        }
        else
        {
          minut[0]++;
          minut[1] = '0';
        }
      }
      else
      {
       minut[1]++;
      }
      secunda[0] = '0';
      secunda[1] = '0';
    }
    else
    {
      secunda[0]++;
      secunda[1] = '0';
    }
  }
  else
  {
    secunda[1]++;
  }
}


int main()      // meniul principal
{
  Serial.begin(9600);  // deschidem conexiunea seriala
  lcd.begin(16, 2); // setam numarul de randuri ale LCD-ului = 16 caractere , 2 randuri
  lcd.print("Initializare"); // inceperea programului este afisata pe ecranul LCD
  lcd.clear(); // curatam ecranul
  sei(); // instrucțiune pentru activarea intreruperilor
  OCR1A = 15625; //OCR1A = 15625 = 0x3D09 -- setam valoarea cu care vom lucra
  TCCR1B |= (1 << CS12) | (1 << WGM12) | (1 << CS10); // setam bitii cs12, cs10 si activam modul CTC
  TIMSK1 |= 1 << OCIE1A; //activam temporizatorul de comparare a întreruperii
  void Analog_to_Digital_Cnoverted_Initialized(); // initializarea registriilor microcontrolerului
  while(1) //functia care ajuta la rularea programului, care e indispensabila rularii
  {
  }
}
