<script>
	export let segment;
</script>

<nav>
    <div class="nav-wrapper">
        <a href="#" class="brand-logo right">Logo</a>
        <ul id="nav-mobile" class="left hide-on-med-and-down">
            <li><a class='{segment === undefined ? "selected" : ""}' href='.'>home</a></li>
            <li><a class='{segment === "states" ? "selected" : ""}' href='states'>states</a></li>
        </ul>
    </div>
</nav>
