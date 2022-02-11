# Angularcrud

## create App

1. ng new my-crud-app --routing

2. npm install bootstrap --save

3. ng generate module post --routing

## components
1. ng generate component post/index
2. ng generate component post/view
3. ng generate component post/create
4. ng generate component post/edit

## Create Route
post-routing.module.ts
```javascript 
const routes: Routes = [
  { path: 'post', redirectTo: 'post/index', pathMatch: 'full'},
  { path: 'post/index', component: IndexComponent },
  { path: 'post/:postId/view', component: ViewComponent },
  { path: 'post/create', component: CreateComponent },
  { path: 'post/:postId/edit', component: EditComponent } 
];
  
@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
export class PostRoutingModule { }
```
## Create Interface
ng generate interface post/post

```javascript
export interface Post {
    id: number;
    title: string;
    body: string;
}
```

## Create Services
ng generate service post/post

```javascript
import { Injectable } from '@angular/core';
import { HttpClient, HttpHeaders } from '@angular/common/http';
   
import {  Observable, throwError } from 'rxjs';
import { catchError } from 'rxjs/operators';
    
import { Post } from './post';
     
@Injectable({
  providedIn: 'root'
})
export class PostService {
     
  private apiURL = "https://jsonplaceholder.typicode.com";
     
  httpOptions = {
    headers: new HttpHeaders({
      'Content-Type': 'application/json'
    })
  }
   
  constructor(private httpClient: HttpClient) { }
     
  getAll(): Observable {

    return this.httpClient.get(this.apiURL + '/posts/')

    .pipe(
      catchError(this.errorHandler)
    )
  }
     
  create(post:Post): Observable {

    return this.httpClient.post(this.apiURL + '/posts/', JSON.stringify(post), this.httpOptions)

    .pipe(
      catchError(this.errorHandler)
    )
  }  
     
  find(id:number): Observable {

    return this.httpClient.get(this.apiURL + '/posts/' + id)

    .pipe(
      catchError(this.errorHandler)
    )
  }
     
  update(id:number, post:Post): Observable {

    return this.httpClient.put(this.apiURL + '/posts/' + id, JSON.stringify(post), this.httpOptions)

    .pipe(
      catchError(this.errorHandler)
    )
  }
     
  delete(id:number){
    return this.httpClient.delete(this.apiURL + '/posts/' + id, this.httpOptions)

    .pipe(
      catchError(this.errorHandler)
    )
  }
    
    
  errorHandler(error:any) {
    let errorMessage = '';
    if(error.error instanceof ErrorEvent) {
      errorMessage = error.error.message;
    } else {
      errorMessage = `Error Code: ${error.status}\nMessage: ${error.message}`;
    }
    return throwError(errorMessage);
 }
}
```
## List Page Template and Component: index

```javascript
import { Component, OnInit } from '@angular/core';
import { PostService } from '../post.service';
import { Post } from '../post';
    
@Component({
  selector: 'app-index',
  templateUrl: './index.component.html',
  styleUrls: ['./index.component.css']
})
export class IndexComponent implements OnInit {
     
  posts: Post[] = [];
   
  constructor(public postService: PostService) { }
    
  ngOnInit(): void {
    this.postService.getAll().subscribe((data: Post[])=>{
      this.posts = data;
      console.log(this.posts);
    })  
  }
    
  deletePost(id:number){
    this.postService.delete(id).subscribe(res => {
         this.posts = this.posts.filter(item => item.id !== id);
         console.log('Post deleted successfully!');
    })
  }
  
}
```

```javascript
<div class="container">
    <h1>Angular 12 CRUD Example - ItSolutionStuff.com</h1>
  
    <a href="#" routerLink="/post/create/" class="btn btn-success">Create New Post</a>
    
    <table class="table table-bordered">
      <tr>
        <th>ID</th>
        <th>Title</th>
        <th>Body</th>
        <th width="220px">Action</th>
      </tr>
      <tr *ngFor="let post of posts">
        <td>{{ post.id }}</td>
        <td>{{ post.title }}</td>
        <td>{{ post.body }}</td>
        <td>
          <a href="#" [routerLink]="['/post/', post.id, 'view']" class="btn btn-info">View</a>
          <a href="#" [routerLink]="['/post/', post.id, 'edit']" class="btn btn-primary">Edit</a>
          <button type="button" (click)="deletePost(post.id)" class="btn btn-danger">Delete</button>
        </td>
      </tr>
    </table>
 </div>
```

## Create Page Template and Component
```javascript
import { Component, OnInit } from '@angular/core';
import { PostService } from '../post.service';
import { Router } from '@angular/router';
import { FormGroup, FormControl, Validators} from '@angular/forms';
   
@Component({
  selector: 'app-create',
  templateUrl: './create.component.html',
  styleUrls: ['./create.component.css']
})
export class CreateComponent implements OnInit {
  
  form: FormGroup;
   
  constructor(
    public postService: PostService,
    private router: Router
  ) { }
  
  ngOnInit(): void {
    this.form = new FormGroup({
      title: new FormControl('', [Validators.required]),
      body: new FormControl('', Validators.required)
    });
  }
   
  get f(){
    return this.form.controls;
  }
    
  submit(){
    console.log(this.form.value);
    this.postService.create(this.form.value).subscribe(res => {
         console.log('Post created successfully!');
         this.router.navigateByUrl('post/index');
    })
  }
  
}
```

```javascript
<div class="container">
    <h1>Create New Post</h1>
  
    <a href="#" routerLink="/post/index" class="btn btn-primary">Back</a>
        
    <form [formGroup]="form" (ngSubmit)="submit()">
  
        <div class="form-group">
            <label for="title">Title:</label>
            <input 
                formControlName="title"
                id="title" 
                type="text" 
                class="form-control">
            <div *ngIf="f.title.touched && f.title.invalid" class="alert alert-danger">
                <div *ngIf="f.title.errors?.required">Title is required.</div>
            </div>
        </div>
  
        <div class="form-group">
            <label for="body">Body</label>
            <textarea 
                formControlName="body"
                id="body" 
                type="text" 
                class="form-control">
            </textarea>
            <div *ngIf="f.body.touched && f.body.invalid" class="alert alert-danger">
                <div *ngIf="f.body.errors?.required">Body is required.</div>
            </div>
        </div>
  
        <button class="btn btn-primary" type="submit" [disabled]="!form.valid">Submit</button>
    </form>
</div>
```

## Edit Page Template and Component
```javascript
import { Component, OnInit } from '@angular/core';
import { PostService } from '../post.service';
import { ActivatedRoute, Router } from '@angular/router';
import { Post } from '../post';
import { FormGroup, FormControl, Validators} from '@angular/forms';
   
@Component({
  selector: 'app-edit',
  templateUrl: './edit.component.html',
  styleUrls: ['./edit.component.css']
})
export class EditComponent implements OnInit {
    
  id: number;
  post: Post;
  form: FormGroup;
  
  constructor(
    public postService: PostService,
    private route: ActivatedRoute,
    private router: Router
  ) { }
  
  ngOnInit(): void {
    this.id = this.route.snapshot.params['postId'];
    this.postService.find(this.id).subscribe((data: Post)=>{
      this.post = data;
    });
    
    this.form = new FormGroup({
      title: new FormControl('', [Validators.required]),
      body: new FormControl('', Validators.required)
    });
  }
   
  get f(){
    return this.form.controls;
  }
     
  submit(){
    console.log(this.form.value);
    this.postService.update(this.id, this.form.value).subscribe(res => {
         console.log('Post updated successfully!');
         this.router.navigateByUrl('post/index');
    })
  }
   
}
```

```javascript
<div class="container">
    <h1>Update Post</h1>
  
    <a href="#" routerLink="/post/index" class="btn btn-primary">Back</a>
        
    <form [formGroup]="form" (ngSubmit)="submit()">
  
        <div class="form-group">
            <label for="title">Title:</label>
            <input 
                formControlName="title"
                id="title" 
                type="text" 
                [(ngModel)]="post.title"
                class="form-control">
            <div *ngIf="f.title.touched && f.title.invalid" class="alert alert-danger">
                <div *ngIf="f.title.errors?.required">Title is required.</div>
            </div>
        </div>
         
        <div class="form-group">
            <label for="body">Body</label>
            <textarea 
                formControlName="body"
                id="body" 
                type="text" 
                [(ngModel)]="post.body"
                class="form-control">
            </textarea>
            <div *ngIf="f.body.touched && f.body.invalid" class="alert alert-danger">
                <div *ngIf="f.body.errors?.required">Body is required.</div>
            </div>
        </div>
        
        <button class="btn btn-primary" type="submit" [disabled]="!form.valid">Update</button>
    </form>
</div>
```

## Detail Page Template and Component
```javascript
import { Component, OnInit } from '@angular/core';
import { PostService } from '../post.service';
import { ActivatedRoute, Router } from '@angular/router';
import { Post } from '../post';
  
@Component({
  selector: 'app-view',
  templateUrl: './view.component.html',
  styleUrls: ['./view.component.css']
})
export class ViewComponent implements OnInit {
   
  id: number;
  post: Post;
   
  constructor(
    public postService: PostService,
    private route: ActivatedRoute,
    private router: Router
   ) { }
  
  ngOnInit(): void {
    this.id = this.route.snapshot.params['postId'];
      
    this.postService.find(this.id).subscribe((data: Post)=>{
      this.post = data;
    });
  }
  
}
```

```javascript
<div class="container">
    <h1>View Post</h1>
  
    <a href="#" routerLink="/post/index" class="btn btn-primary">Back</a>
   
    <div>
        <strong>ID:</strong>
        <p>{{ post.id }}</p>
    </div>
   
    <div>
        <strong>Title:</strong>
        <p>{{ post.title }}</p>
    </div>
   
    <div>
        <strong>Body:</strong>
        <p>{{ post.body }}</p>
    </div>
   
</div>
```

##  update app.component.html
<router-outlet></router-outlet>
