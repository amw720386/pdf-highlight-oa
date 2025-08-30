# PDF Highlighter 

## Project Overview

This project is a PDF viewer and keyword search application developed as part of the Adanomad Tech Consulting Challenge. It allows users to upload PDF documents, view them in a web browser, search for keywords, and highlight matching text.

# Implementations

Firstly before I start I'd like apologize for how long I took to solve this OA, on top of starting late I also managed to convince myself that the future improvements section here is what I had to implement. Let's just say, attempting to have a drawable canvas over a PDF while using react-pdf-highlighter and maintaining mobile-friendly code is **NOT** fun. 

1. Multi-pdf support with keyword search supported amongst both.

https://github.com/user-attachments/assets/244404b6-5372-4829-8c3a-316dadec60c4

- Every uploaded PDF is tracked in a docs map ({pdfId → {name, fileUrl, ocrUrl}}).

- On search, the app iterates all docs, runs searchPdf(keywords, fileUrl) (falls back to ocrUrl if no text layer), and aggregates all matches into one highlights list.

- Each hit is tagged with its owning pdfId (highlightOwner[id] = pdfId), so clicking a result auto-switches to that PDF and scrolls to the rect.

- If keywords find nothing (or query starts with ~), it uses page-level semantic hits to pick a page, then re-localizes with a tiny keyword search on that page.


2. Searchable embedded vectors

https://github.com/user-attachments/assets/82b9d6c4-751f-4e89-be80-4ed2c72f96ca

Do note that this is simply a very rough implementation of indexing/implementing searchable-vectors, obviously with a more precise model and more queries to the model, it'd be a LOT more precise. Do also note that this feature and the next one are both very intertwined and reflective of each other. 

- On upload, we extract text per page (prefer the PDF text layer; OCR only if needed), then create page embeddings with text-embedding-3-small (1536-d).

- Those vectors are stored locally in SQLite alongside the page text and pdfId.

- On search, we embed the query once, compute cosine similarity against all stored page vectors, and return the top pages.

- For each hit page, we pull a few probe words from its text to quickly re-locate and draw an exact highlight in the viewer.


3. Page level indexing

- We index one vector per page: on upload we extract each page’s text (use the PDF text layer when present; if a page is image-only, we OCR just that page) and store {pdfId, page, text, embedding} in SQLite.

- Page-level keeps vectors small and localized, so huge PDFs are searchable without giant, single-document embeddings.

- On search, we embed the query once, compute cosine similarity against all page vectors, and return the best pages.

- For each hit page, we grab a few probe words from its text to quickly re-locate and draw an exact highlight in the viewer—even when the original PDF was entirely images.


4. PDF upload to supabase and relevant schema


https://github.com/user-attachments/assets/76e682d7-597e-465c-aa33-256ddd64628c

Note that there are several uploads of the same document (the drylabs one since I used it several times across these demos). This is for possible future implementation of version control/several "sets" of annotations for one document. 

- Upload: On file drop, we create a pdfId and upload the PDF to Supabase Storage (pdfs/uploads/{pdfId}/…); a blob URL is used for instant preview.

- Highlights schema (DB): Supabase table highlights(id text, pdf_id text, data jsonb, created_at, updated_at); routes /api/highlight/get|update read/write it.

- Search index (local): SQLite table page_embeddings(pdfId, page, text, embedding-json) built on upload (page text or OCR → per-page embedding); /api/semantic/index|query manage/query it.

- Result: PDFs are stored, highlights persisted, and large/image-only docs are searchable via page-level embeddings.


**Everything below this point is from the original README.md**

## Features

- PDF document upload and display
- Page navigation (next, previous, jump to specific page)
- Zoom in/out functionality
- Document information display (total pages, current page)
- Keyword search across the entire PDF
- Text highlighting for search matches
- Sidebar for search results and navigation
- Responsive design for various screen sizes
- Persistent storage of highlights using SQLite or Supabase

## Technologies Used

- Next.js
- React 
- TypeScript
- react-pdf library for PDF rendering
- Tailwind CSS for stylinge
- SQLite for local highlight storage
- Supabase for cloud-based highlight storage (optional)

## Getting Started

1. Clone the repository
2. Install dependencies: `pnpm install`
3. Run the development server: `pnpm run dev`
4. Open [http://localhost:3000](http://localhost:3000) in your browser

## Project Structure

- `app/page.js`: Main entry point of the application
- `app/components/`: React components for various parts of the application
- `app/utils/`: Utility functions for PDF processing and highlight storage
- `app/styles/`: CSS files for styling
- `app/api/`: API routes for handling highlight operations

## Key Components

- `App.tsx`: Core application component
- `PdfViewer.tsx`: Handles PDF rendering and navigation
- `KeywordSearch.tsx`: Manages keyword search functionality
- `HighlightPopup.tsx`: Displays information about highlighted text
- `Sidebar.tsx`: Shows search results and navigation options
- `highlightStorage.ts`: Manages highlight storage operations
- `sqliteUtils.ts`: Handles SQLite database operations

## Features

- Has a highlight storage system supporting both SQLite and Supabase
- API routes for creating, retrieving, updating, and deleting highlights
- User authentication and document permissions (currently disabled)
- Export/import as JSON functionality for highlights
- Scroll the sidebar highlighted area into view across different PDFs. 


## Future Improvements

- Implement annotation tools (e.g., freehand drawing, text notes)
- Add support for multiple document search
- Pre-process batch PDFs for quicker highlights
- Enhance mobile responsiveness for better small-screen experience
- Optimize performance for large PDF files
- Upload the PDF into the database.

## License

[MIT License](https://opensource.org/licenses/MIT)

## Acknowledgements

- [Next.js](https://nextjs.org/) for the React framework
- [SQLite](https://www.sqlite.org/) for local database storage
- [Supabase](https://supabase.io/) for cloud database capabilities
- [react-pdf](https://github.com/wojtekmaj/react-pdf) for PDF rendering capabilities
- [Tailwind CSS](https://tailwindcss.com/) for utility-first CSS framework
