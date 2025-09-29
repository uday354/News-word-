<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Word News</title>
  <style>
    body { font-family: Arial, sans-serif; background: #f4f4f9; margin: 0; padding: 0; }
    header { background: linear-gradient(to right, red, orange, green, blue, purple); 
             color: white; padding: 20px; text-align: center; font-size: 28px; font-weight: bold; }
    main { padding: 20px; max-width: 900px; margin: auto; }
    input, textarea { width: 100%; padding: 10px; margin: 10px 0; border-radius: 5px; border: 1px solid #ccc; }
    button { padding: 8px 15px; margin: 5px; border: none; border-radius: 5px; cursor: pointer; }
    .save-btn { background: #4CAF50; color: white; }
    .save-btn:hover { background: #45a049; }
    .delete-btn { background: #e74c3c; color: white; }
    .delete-btn:hover { background: #c0392b; }
    .share-btn { background: #3498db; color: white; }
    .share-btn:hover { background: #2980b9; }
    .comment-btn { background: #f39c12; color: white; }
    .comment-btn:hover { background: #d35400; }
    .post { background: white; padding: 15px; margin: 15px 0; border-radius: 8px; 
            box-shadow: 0 2px 5px rgba(0,0,0,0.1); }
    .media { margin-top: 10px; }
    .media img, .media video { max-width: 100%; margin-top: 10px; border-radius: 8px; }
    .comments { margin-top: 10px; padding-left: 15px; font-size: 14px; color: #333; }
    h2 { margin-top: 40px; }
  </style>
</head>
<body>
  <header>
    🌍 Word News  
    <button onclick="followUser()">⭐ Follow</button> 
    Followers: <span id="followersCount">0</span>
  </header>

  <main>
    <h2>Create a Post</h2>
    <input type="text" id="title" placeholder="Post Title">
    <textarea id="content" rows="5" placeholder="Write your news here..."></textarea>
    <input type="file" id="media" accept="image/*,video/*">
    <button class="save-btn" onclick="savePost()">Publish</button>

    <h2>All Posts</h2>
    <div id="posts"></div>
  </main>

  
<script type="module">
  import { createClient } from "https://esm.sh/@supabase/supabase-js";

  // 🔑 Supabase project details
  const SUPABASE_URL = "https://stdnepkusdzqexhmuahz.supabase.co";
  const SUPABASE_KEY = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InN0ZG5lcGt1c2R6cWV4aG11YWh6Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3NTkwNDE1OTAsImV4cCI6MjA3NDYxNzU5MH0.ysXwofHVWJtiE86r6dEv1NQLp30OXVlE4KUMz3c3IsQ";
  const supabase = createClient(SUPABASE_URL, SUPABASE_KEY);

  // -------- FOLLOW SYSTEM --------
  async function loadFollowers() {
    let { data, error } = await supabase.from("profile").select("followers").eq("id", 1).single();
    if (!error && data) {
      document.getElementById("followersCount").innerText = data.followers;
    }
  }

  async function followUser() {
    let { data, error } = await supabase.from("profile").select("followers").eq("id", 1).single();
    if (!error && data) {
      let newCount = (data.followers || 0) + 1;
      await supabase.from("profile").update({ followers: newCount }).eq("id", 1);
      document.getElementById("followersCount").innerText = newCount;
    }
  }

  // -------- POSTS SYSTEM --------
  async function loadPosts() {
    const { data, error } = await supabase
      .from("posts")
      .select("*")
      .order("id", { ascending: false });

    if (error) {
      console.error("Error loading posts:", error);
      return;
    }

    let container = document.getElementById("posts");
    container.innerHTML = "";

    data.forEach((p) => {
      let div = document.createElement("div");
      div.className = "post";
      let mediaHTML = "";

      if (p.media && p.mediaType === "image") {
        mediaHTML = `<div class="media"><img src="${p.media}" alt="Image"></div>`;
      } else if (p.media && p.mediaType === "video") {
        mediaHTML = `<div class="media"><video controls src="${p.media}"></video></div>`;
      }

      div.innerHTML = `
        <h3>${p.title}</h3>
        <p>${p.content}</p>
        ${mediaHTML}
        <button onclick="likePost(${p.id})">👍 Like (${p.likes || 0})</button>
        <button class="delete-btn" onclick="deletePost(${p.id})">🗑 Delete</button>
        <button class="share-btn" onclick="sharePost('${p.title}', '${p.content}')">📤 Share</button>
        <button class="comment-btn" onclick="addComment(${p.id})">💬 Comment</button>
        <div class="comments">
          ${(p.comments || []).map(c => `<p>👉 ${c}</p>`).join("")}
        </div>
      `;
      container.appendChild(div);
    });
  }

  async function savePost() {
    let title = document.getElementById("title").value;
    let content = document.getElementById("content").value;
    let fileInput = document.getElementById("media").files[0];

    if (!title || !content) {
      alert("Please fill all fields!");
      return;
    }

    if (fileInput) {
      let reader = new FileReader();
      reader.onload = async function (e) {
        let mediaData = e.target.result;
        let mediaType = fileInput.type.startsWith("image") ? "image" : "video";

        await supabase.from(\"posts\").insert([
          { title, content, media: mediaData, mediaType: mediaType, likes: 0 }
        ]);
        alert('✅ Post uploaded successfully!');
        loadPosts();
      };
      reader.readAsDataURL(fileInput);
    } else {
      await supabase.from(\"posts\").insert([{ title, content, likes: 0 }]);
        alert('✅ Post uploaded successfully!');
      loadPosts();
    }

    document.getElementById("title").value = "";
    document.getElementById("content").value = "";
    document.getElementById("media").value = "";
  }

  async function deletePost(id) {
    await supabase.from("posts").delete().eq("id", id);
    loadPosts();
  }

  async function likePost(id) {
    let { data } = await supabase.from("posts").select("likes").eq("id", id).single();
    let newLikes = (data.likes || 0) + 1;
    await supabase.from("posts").update({ likes: newLikes }).eq("id", id);
    loadPosts();
  }

  async function addComment(id) {
    let comment = prompt("Write your comment:");
    if (comment) {
      let { data } = await supabase.from("posts").select("comments").eq("id", id).single();
      let comments = data.comments || [];
      comments.push(comment);

      await supabase.from("posts").update({ comments }).eq("id", id);
      loadPosts();
    }
  }

  function sharePost(title, content) {
    let text = `${title}

${content}`;
    if (navigator.share) {
      navigator.share({ title: "Word News", text })
        .catch(err => console.log("Share canceled", err));
    } else {
      navigator.clipboard.writeText(text);
      alert("Post copied! Paste it anywhere.");
    }
  }

  // Global functions
  window.savePost = savePost;
  window.deletePost = deletePost;
  window.addComment = addComment;
  window.sharePost = sharePost;
  window.likePost = likePost;
  window.followUser = followUser;

  // Initial load
  loadFollowers();
  loadPosts();
</script>

</body>
</html>
