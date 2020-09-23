## ESPHome Effects

Shamelessly adapted from https://www.tweaking4all.com/hardware/arduino/adruino-led-strip-effects/

'''yaml
- addressable_lambda:
    name: "Fire"
    update_interval: 15ms
    lambda: |-
      static byte heat[255]; //Change this so that it is greater than the number of pixels
      static int Cooling = 55;
      static int Sparking = 120;

      int cooldown;
    
      // Step 1.  Cool down every cell a little
      for( int i = 0; i < it.size(); i++) {
        cooldown = random(0, ((Cooling * 10) / it.size()) + 2);
      
        if(cooldown>heat[i]) {
          heat[i]=0;
        } else {
          heat[i]=heat[i]-cooldown;
        }
      }
    
      // Step 2.  Heat from each cell drifts 'up' and diffuses a little
      for( int k= it.size() - 1; k >= 2; k--) {
        heat[k] = (heat[k - 1] + heat[k - 2] + heat[k - 2]) / 3;
      }
      
      // Step 3.  Randomly ignite new 'sparks' near the bottom
      if( random(255) < Sparking ) {
        int y = random(7);
        heat[y] = heat[y] + random(160,255);
      }

      // Step 4.  Convert heat to LED colors
      for( int j = 0; j < it.size(); j++) {

          byte t192 = round((heat[j]/255.0)*191);
 
          // calculate ramp up from
          byte heatramp = t192 & 0x3F; // 0..63
          heatramp <<= 2; // scale up to 0..252
        
          // figure out which third of the spectrum we're in:
          if( t192 > 0x80) {                     // hottest
            it[j] = light::ESPColor(255, 255, heatramp);
          } else if( t192 > 0x40 ) {             // middle
            it[j] = light::ESPColor(255, heatramp, 0);
          } else {                               // coolest
            it[j] = light::ESPColor(heatramp, 0, 0);
          }
      }
'''