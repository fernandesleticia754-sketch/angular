import { Component } from '@angular/core';
import { DomSanitizer, SafeUrl } from '@angular/platform-browser';

@Component({
  selector: 'app-root',
  template: `
    <div class="h-screen w-full flex items-center justify-center p-4 bg-zinc-800">
      <div class="bg-white p-4 rounded-xl shadow-lg transform rotate-[-2deg]">
        <div class="relative w-full aspect-[2/3] overflow-hidden rounded-md">
          <img
            *ngIf="generatedImage"
            [src]="generatedImage"
            alt="Imagem estilo Polaroid"
            class="w-full h-full object-cover"
          />
        </div>
        <div class="p-2 text-center text-zinc-800 text-lg font-serif">
          Momento
        </div>
      </div>
    </div>
  `,
  styles: [`
    @import url('https://fonts.googleapis.com/css2?family=Shadows+Into+Light&display=swap');
    
    .font-serif {
      font-family: 'Shadows Into Light', cursive;
    }
  `],
  standalone: true,
})
export class App {
  generatedImage: SafeUrl | null = null;

  constructor(private sanitizer: DomSanitizer) {
    this.generateImage();
  }

  async generateImage() {
    const userPrompt = "Uma foto tirada com uma câmera Polaroid. A foto deve parecer uma foto normal, sem um assunto ou propriedade clara. A foto deve ter um leve efeito de desfoque e uma fonte de luz consistente, como um flash de uma sala escura, espalhada por toda a foto. O fundo atrás das duas pessoas é uma cortina branca. Eles estão se abraçando.";

    const payload = {
      contents: [{
        parts: [{ text: userPrompt }]
      }]
    };

    const apiKey = "SUA_CHAVE_API_AQUI"; // coloque em environment.ts depois
    const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-image-preview:generateContent?key=${apiKey}`;

    try {
      const response = await fetch(apiUrl, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload)
      });

      const result = await response.json();
      const base64Data = result?.candidates?.[0]?.content?.parts?.[0]?.inlineData?.data;

      if (base64Data) {
        const imageUrl = `data:image/png;base64,${base64Data}`;
        this.generatedImage = this.sanitizer.bypassSecurityTrustUrl(imageUrl);
      }
    } catch (error) {
      console.error('Erro ao gerar imagem:', error);
    }
  }
}
