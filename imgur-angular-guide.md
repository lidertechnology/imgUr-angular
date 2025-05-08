# Guía completa: Integración de Angular Standalone con Imgur API

## Índice
1. [Registro en Imgur para desarrolladores](#1-registro-en-imgur-para-desarrolladores)
2. [Configuración del proyecto Angular](#2-configuración-del-proyecto-angular)
3. [Creación del servicio de Imgur](#3-creación-del-servicio-de-imgur)
4. [Implementación del componente para cargar imágenes](#4-implementación-del-componente-para-cargar-imágenes)
5. [Implementación del componente para mostrar imágenes](#5-implementación-del-componente-para-mostrar-imágenes)
6. [Integración en la aplicación principal](#6-integración-en-la-aplicación-principal)
7. [Pruebas y depuración](#7-pruebas-y-depuración)

## 1. Registro en Imgur para desarrolladores

### 1.1 Crear una cuenta en Imgur
1. Visita [Imgur](https://imgur.com/) y crea una cuenta si aún no tienes una.
2. Inicia sesión con tu cuenta.

### 1.2 Registrar una aplicación
1. Ve a [Imgur API Applications](https://api.imgur.com/oauth2/addclient).
2. Completa el formulario de registro:
   - **Application name**: Nombre de tu aplicación
   - **Authorization type**: Selecciona "OAuth 2 authorization without a callback URL" para una aplicación simple
   - **Email**: Tu correo electrónico
   - **Description**: Breve descripción de tu aplicación
   - **Authorization callback URL**: Puedes usar `http://localhost:4200` para desarrollo local

3. Haz clic en "Submit" para registrar tu aplicación.
4. Imgur te proporcionará un **Client ID** y un **Client Secret**. Guarda estos valores de forma segura, los necesitarás más adelante.

## 2. Configuración del proyecto Angular

### 2.1 Crear un nuevo proyecto Angular Standalone (o usar uno existente)
```bash
# Instalar Angular CLI si aún no lo tienes
npm install -g @angular/cli

# Crear un nuevo proyecto con la estructura standalone
ng new imgur-angular-app --standalone
cd imgur-angular-app
```

### 2.2 Instalar las dependencias necesarias
```bash
npm install @angular/common/http
```

### 2.3 Configurar el entorno para guardar las credenciales de la API

Crea o actualiza el archivo `src/environments/environment.ts`:

```typescript
export const environment = {
  production: false,
  imgurApi: {
    clientId: 'TU_CLIENT_ID', // Reemplaza con tu Client ID de Imgur
    apiUrl: 'https://api.imgur.com/3'
  }
};
```

También actualiza o crea `src/environments/environment.prod.ts` para producción:

```typescript
export const environment = {
  production: true,
  imgurApi: {
    clientId: 'TU_CLIENT_ID', // Reemplaza con tu Client ID de Imgur
    apiUrl: 'https://api.imgur.com/3'
  }
};
```

## 3. Creación del servicio de Imgur

### 3.1 Crear un modelo para las imágenes de Imgur

Crea un archivo `src/app/models/imgur-image.model.ts`:

```typescript
export interface ImgurImage {
  id: string;
  title: string | null;
  description: string | null;
  datetime: number;
  type: string;
  animated: boolean;
  width: number;
  height: number;
  size: number;
  views: number;
  bandwidth: number;
  link: string;
  favorite: boolean;
  nsfw: boolean | null;
  section: string | null;
  account_url: string | null;
  account_id: number | null;
  is_ad: boolean;
  in_most_viral: boolean;
  has_sound: boolean;
  tags: string[];
  ad_type: number;
  ad_url: string;
  edited: string;
  in_gallery: boolean;
  deletehash: string;
  name: string;
}

export interface ImgurUploadResponse {
  data: ImgurImage;
  success: boolean;
  status: number;
}

export interface ImgurAlbumResponse {
  data: ImgurImage[];
  success: boolean;
  status: number;
}
```

### 3.2 Crear un servicio para interactuar con la API de Imgur

Crea un archivo `src/app/services/imgur.service.ts`:

```typescript
import { Injectable, inject } from '@angular/core';
import { HttpClient, HttpHeaders } from '@angular/common/http';
import { Observable } from 'rxjs';
import { environment } from '../../environments/environment';
import { ImgurUploadResponse, ImgurAlbumResponse } from '../models/imgur-image.model';

@Injectable({
  providedIn: 'root'
})
export class ImgurService {
  private http = inject(HttpClient);
  private apiUrl = environment.imgurApi.apiUrl;
  private clientId = environment.imgurApi.clientId;

  // Método para subir una imagen desde un archivo
  uploadImage(image: File): Observable<ImgurUploadResponse> {
    const formData = new FormData();
    formData.append('image', image);

    return this.http.post<ImgurUploadResponse>(`${this.apiUrl}/upload`, formData, {
      headers: new HttpHeaders({
        'Authorization': `Client-ID ${this.clientId}`
      })
    });
  }

  // Método para subir una imagen desde una URL
  uploadImageUrl(imageUrl: string): Observable<ImgurUploadResponse> {
    const formData = new FormData();
    formData.append('image', imageUrl);

    return this.http.post<ImgurUploadResponse>(`${this.apiUrl}/upload`, formData, {
      headers: new HttpHeaders({
        'Authorization': `Client-ID ${this.clientId}`
      })
    });
  }

  // Método para obtener imágenes de un álbum específico
  getAlbumImages(albumHash: string): Observable<ImgurAlbumResponse> {
    return this.http.get<ImgurAlbumResponse>(`${this.apiUrl}/album/${albumHash}/images`, {
      headers: new HttpHeaders({
        'Authorization': `Client-ID ${this.clientId}`
      })
    });
  }

  // Método para obtener información de una imagen específica
  getImage(imageId: string): Observable<ImgurUploadResponse> {
    return this.http.get<ImgurUploadResponse>(`${this.apiUrl}/image/${imageId}`, {
      headers: new HttpHeaders({
        'Authorization': `Client-ID ${this.clientId}`
      })
    });
  }

  // Método para eliminar una imagen
  deleteImage(deleteHash: string): Observable<any> {
    return this.http.delete(`${this.apiUrl}/image/${deleteHash}`, {
      headers: new HttpHeaders({
        'Authorization': `Client-ID ${this.clientId}`
      })
    });
  }
}
```

### 3.3 Configurar el HttpClientModule en la aplicación

Actualiza tu archivo `src/app/app.config.ts`:

```typescript
import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient } from '@angular/common/http';

import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideHttpClient()
  ]
};
```

## 4. Implementación del componente para cargar imágenes

### 4.1 Crear un componente para cargar imágenes

```bash
ng generate component components/image-upload --standalone
```

### 4.2 Implementar el componente de carga de imágenes

Actualiza el archivo `src/app/components/image-upload/image-upload.component.ts`:

```typescript
import { Component, EventEmitter, Output, inject } from '@angular/core';
import { CommonModule } from '@angular/common';
import { FormsModule } from '@angular/forms';
import { ImgurService } from '../../services/imgur.service';
import { ImgurImage } from '../../models/imgur-image.model';

@Component({
  selector: 'app-image-upload',
  standalone: true,
  imports: [CommonModule, FormsModule],
  templateUrl: './image-upload.component.html',
  styleUrls: ['./image-upload.component.css']
})
export class ImageUploadComponent {
  @Output() imageUploaded = new EventEmitter<ImgurImage>();
  
  selectedFile: File | null = null;
  isUploading = false;
  errorMessage = '';
  
  private imgurService = inject(ImgurService);

  onFileSelected(event: Event): void {
    const input = event.target as HTMLInputElement;
    if (input.files && input.files.length > 0) {
      this.selectedFile = input.files[0];
    }
  }

  upload(): void {
    if (!this.selectedFile) {
      this.errorMessage = 'Por favor, selecciona un archivo primero';
      return;
    }

    this.isUploading = true;
    this.errorMessage = '';

    this.imgurService.uploadImage(this.selectedFile).subscribe({
      next: (response) => {
        this.isUploading = false;
        if (response.success) {
          this.imageUploaded.emit(response.data);
          this.selectedFile = null;
        } else {
          this.errorMessage = 'Error al subir la imagen';
        }
      },
      error: (error) => {
        this.isUploading = false;
        this.errorMessage = 'Error al subir la imagen: ' + (error.message || 'Desconocido');
        console.error('Error al subir imagen a Imgur:', error);
      }
    });
  }
}
```

### 4.3 Crear la plantilla HTML del componente

Crea el archivo `src/app/components/image-upload/image-upload.component.html`:

```html
<div class="image-upload-container">
  <h3>Subir imagen a Imgur</h3>
  
  <div class="file-input">
    <input type="file" id="fileInput" accept="image/*" (change)="onFileSelected($event)">
    <label for="fileInput">
      <span *ngIf="selectedFile">{{ selectedFile.name }}</span>
      <span *ngIf="!selectedFile">Seleccionar archivo</span>
    </label>
  </div>
  
  <button 
    [disabled]="!selectedFile || isUploading" 
    (click)="upload()" 
    class="upload-button">
    <span *ngIf="!isUploading">Subir imagen</span>
    <span *ngIf="isUploading">Subiendo...</span>
  </button>
  
  <div *ngIf="errorMessage" class="error-message">
    {{ errorMessage }}
  </div>
</div>
```

### 4.4 Estilos CSS para el componente

Actualiza el archivo `src/app/components/image-upload/image-upload.component.css`:

```css
.image-upload-container {
  max-width: 500px;
  margin: 20px auto;
  padding: 20px;
  border-radius: 8px;
  background-color: #f9f9f9;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

h3 {
  margin-top: 0;
  color: #333;
}

.file-input {
  margin-bottom: 15px;
}

.file-input input[type="file"] {
  display: none;
}

.file-input label {
  display: block;
  padding: 10px 15px;
  background-color: #e9e9e9;
  border: 1px solid #ddd;
  border-radius: 4px;
  cursor: pointer;
  text-overflow: ellipsis;
  white-space: nowrap;
  overflow: hidden;
}

.file-input label:hover {
  background-color: #e1e1e1;
}

.upload-button {
  display: block;
  width: 100%;
  padding: 10px 15px;
  background-color: #0d66d0;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-weight: bold;
}

.upload-button:hover:not(:disabled) {
  background-color: #0b57b3;
}

.upload-button:disabled {
  background-color: #7cacde;
  cursor: not-allowed;
}

.error-message {
  margin-top: 15px;
  color: #d03030;
  font-size: 14px;
}
```

## 5. Implementación del componente para mostrar imágenes

### 5.1 Crear un componente para mostrar imágenes

```bash
ng generate component components/image-gallery --standalone
```

### 5.2 Implementar el componente de galería de imágenes

Actualiza el archivo `src/app/components/image-gallery/image-gallery.component.ts`:

```typescript
import { Component, Input, OnInit, inject } from '@angular/core';
import { CommonModule } from '@angular/common';
import { ImgurService } from '../../services/imgur.service';
import { ImgurImage } from '../../models/imgur-image.model';

@Component({
  selector: 'app-image-gallery',
  standalone: true,
  imports: [CommonModule],
  templateUrl: './image-gallery.component.html',
  styleUrls: ['./image-gallery.component.css']
})
export class ImageGalleryComponent implements OnInit {
  @Input() albumHash: string = '';
  @Input() images: ImgurImage[] = [];

  isLoading = false;
  errorMessage = '';
  
  private imgurService = inject(ImgurService);

  ngOnInit(): void {
    // Si se proporciona un albumHash, cargamos las imágenes del álbum
    if (this.albumHash) {
      this.loadAlbumImages();
    }
  }

  loadAlbumImages(): void {
    this.isLoading = true;
    this.errorMessage = '';

    this.imgurService.getAlbumImages(this.albumHash).subscribe({
      next: (response) => {
        this.isLoading = false;
        if (response.success) {
          this.images = response.data;
        } else {
          this.errorMessage = 'Error al cargar las imágenes del álbum';
        }
      },
      error: (error) => {
        this.isLoading = false;
        this.errorMessage = 'Error al cargar las imágenes: ' + (error.message || 'Desconocido');
        console.error('Error al cargar imágenes de Imgur:', error);
      }
    });
  }

  addImage(image: ImgurImage): void {
    // Agregamos la imagen recién subida al inicio del array
    this.images = [image, ...this.images];
  }

  deleteImage(index: number): void {
    const image = this.images[index];
    if (image && image.deletehash) {
      this.imgurService.deleteImage(image.deletehash).subscribe({
        next: (response) => {
          if (response.success) {
            this.images = this.images.filter((_, i) => i !== index);
          } else {
            this.errorMessage = 'Error al eliminar la imagen';
          }
        },
        error: (error) => {
          this.errorMessage = 'Error al eliminar la imagen: ' + (error.message || 'Desconocido');
          console.error('Error al eliminar imagen de Imgur:', error);
        }
      });
    }
  }
}
```

### 5.3 Crear la plantilla HTML para la galería

Crea el archivo `src/app/components/image-gallery/image-gallery.component.html`:

```html
<div class="gallery-container">
  <h3>Galería de imágenes</h3>

  <div *ngIf="isLoading" class="loading">
    Cargando imágenes...
  </div>

  <div *ngIf="errorMessage" class="error-message">
    {{ errorMessage }}
  </div>

  <div *ngIf="!isLoading && images.length === 0" class="no-images">
    No hay imágenes para mostrar
  </div>

  <div class="image-grid" *ngIf="images.length > 0">
    <div class="image-card" *ngFor="let image of images; let i = index">
      <div class="image-container">
        <img [src]="image.link" [alt]="image.title || 'Imagen sin título'" loading="lazy">
      </div>
      <div class="image-info">
        <h4>{{ image.title || 'Sin título' }}</h4>
        <p *ngIf="image.description">{{ image.description }}</p>
        <div class="image-meta">
          <span>{{ image.width }}x{{ image.height }}</span>
          <span>{{ image.views }} vistas</span>
        </div>
        <button class="delete-button" (click)="deleteImage(i)" *ngIf="image.deletehash">
          Eliminar
        </button>
      </div>
    </div>
  </div>
</div>
```

### 5.4 Estilos CSS para la galería

Actualiza el archivo `src/app/components/image-gallery/image-gallery.component.css`:

```css
.gallery-container {
  margin: 20px auto;
  padding: 20px;
  border-radius: 8px;
  background-color: #f9f9f9;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

h3 {
  margin-top: 0;
  color: #333;
}

.loading {
  text-align: center;
  padding: 20px;
  color: #666;
}

.error-message {
  color: #d03030;
  padding: 15px;
  background-color: #ffe6e6;
  border-radius: 4px;
  margin-bottom: 15px;
}

.no-images {
  text-align: center;
  padding: 30px;
  color: #666;
  font-style: italic;
}

.image-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 20px;
}

.image-card {
  border-radius: 6px;
  overflow: hidden;
  background-color: white;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s;
}

.image-card:hover {
  transform: translateY(-5px);
}

.image-container {
  height: 200px;
  overflow: hidden;
}

.image-container img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.3s;
}

.image-container img:hover {
  transform: scale(1.05);
}

.image-info {
  padding: 15px;
}

.image-info h4 {
  margin-top: 0;
  margin-bottom: 8px;
  font-size: 16px;
  color: #333;
}

.image-info p {
  margin: 0 0 10px;
  font-size: 14px;
  color: #666;
}

.image-meta {
  display: flex;
  justify-content: space-between;
  font-size: 12px;
  color: #888;
  margin-bottom: 10px;
}

.delete-button {
  display: block;
  width: 100%;
  padding: 8px;
  background-color: #f44336;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

.delete-button:hover {
  background-color: #d32f2f;
}
```

## 6. Integración en la aplicación principal

### 6.1 Actualiza el componente principal de la aplicación

Modifica el archivo `src/app/app.component.ts`:

```typescript
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterOutlet } from '@angular/router';
import { ImageUploadComponent } from './components/image-upload/image-upload.component';
import { ImageGalleryComponent } from './components/image-gallery/image-gallery.component';
import { ImgurImage } from './models/imgur-image.model';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [CommonModule, RouterOutlet, ImageUploadComponent, ImageGalleryComponent],
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'Imgur Angular Integration';
  uploadedImages: ImgurImage[] = [];

  onImageUploaded(image: ImgurImage): void {
    // Agregar la imagen recién subida al array de imágenes
    this.uploadedImages = [image, ...this.uploadedImages];
  }
}
```

### 6.2 Actualiza la plantilla HTML principal

Modifica el archivo `src/app/app.component.html`:

```html
<div class="app-container">
  <header>
    <h1>{{ title }}</h1>
    <p>Integración de Angular Standalone con la API de Imgur</p>
  </header>

  <main>
    <app-image-upload (imageUploaded)="onImageUploaded($event)"></app-image-upload>
    
    <app-image-gallery [images]="uploadedImages"></app-image-gallery>
  </main>

  <footer>
    <p>Desarrollado con Angular y la API de Imgur</p>
  </footer>
</div>
```

### 6.3 Estilos CSS para la aplicación principal

Modifica el archivo `src/app/app.component.css`:

```css
.app-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

header {
  text-align: center;
  margin-bottom: 30px;
}

header h1 {
  margin-bottom: 10px;
  color: #333;
}

header p {
  color: #666;
}

main {
  display: flex;
  flex-direction: column;
  gap: 30px;
}

footer {
  margin-top: 40px;
  text-align: center;
  color: #888;
  font-size: 14px;
  padding-top: 20px;
  border-top: 1px solid #eee;
}
```

## 7. Pruebas y depuración

### 7.1 Preparar la aplicación para pruebas

1. Asegúrate de haber reemplazado 'TU_CLIENT_ID' en el archivo de entorno con tu Client ID real de Imgur.
2. Inicia el servidor de desarrollo:

```bash
ng serve
```

3. Abre tu navegador en `http://localhost:4200`

### 7.2 Verificar la funcionalidad de carga

1. Selecciona una imagen local utilizando el componente de carga.
2. Haz clic en "Subir imagen".
3. Verifica que la imagen aparezca en la galería después de subirse correctamente.

### 7.3 Depuración de problemas comunes

- **Problema**: Error 403 Forbidden al intentar subir imágenes.
  **Solución**: Verifica que el Client ID sea correcto y que hayas registrado correctamente tu aplicación en Imgur.

- **Problema**: Las imágenes no aparecen después de subirlas.
  **Solución**: Verifica la consola del navegador para errores. Asegúrate de que el evento `imageUploaded` está correctamente propagado y capturado.

- **Problema**: Tipos de archivo no aceptados.
  **Solución**: Asegúrate de que estás seleccionando formatos de imagen compatibles con Imgur (JPEG, PNG, GIF, etc.).

### 7.4 Mejoras y optimizaciones opcionales

- Implementa la paginación para mostrar muchas imágenes.
- Agrega soporte para subir múltiples imágenes a la vez.
- Implementa una vista previa de la imagen antes de subirla.
- Agrega funcionalidad de arrastrar y soltar para la carga de imágenes.
- Implementa autenticación OAuth 2.0 para acceder a más funcionalidades de la API de Imgur.

¡Felicidades! Has completado la integración de tu aplicación Angular Standalone con la API de Imgur. Ahora tu aplicación puede cargar imágenes a Imgur y mostrarlas en una galería.
