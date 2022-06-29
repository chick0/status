<script>
    import { parse } from "marked";
    import { github, contact } from "./config.js";

    let isLoaded = false;
    let statusList = [];

    $: if (isLoaded === true) {
        setTimeout(() => {
            document.querySelectorAll("span.tag.label").forEach((element) => {
                const color = window
                    .getComputedStyle(element)
                    .backgroundColor.replace(/[^\d.]/g, " ")
                    .split(" ")
                    .filter((x) => x.length != 0);

                // @ts-ignore
                element.style.color =
                    Math.round(
                        (parseInt(color[0]) * 299 +
                            parseInt(color[1]) * 587 +
                            parseInt(color[2]) * 114) /
                            1000
                    ) > 128
                        ? "#000000"
                        : "#FFFFFF";
            });
        }, 100);
    }

    fetch(`https://api.github.com/repos/${github.repo}/issues`, {
        headers: {
            Accept: "application/vnd.github.v3+json",
        },
    })
        .then((resp) => resp.json())
        .then((json) => {
            statusList = json.filter((e) => {
                let date = new Date(e.created_at);
                e.parsedDate = date.toLocaleString();
                return github.user_id.includes(e.user.login);
            });

            isLoaded = true;
        });
</script>

<section class="section mb-6">
    <div class="container">
        <h1 class="title is-1">서버 상태</h1>
        <p class="subtitle">알려진 문제점들을 확인할 수 있습니다.</p>

        <div class="block">
            {#if contact.email.length != 0}
                <a
                    class="button is-primary is-medium"
                    href="mailto:{contact.email}">이메일</a>
            {/if}
        </div>

        {#if isLoaded === false}
            <div class="notification is-link is-light">
                <p>서버 정보를 불러오고 있습니다.</p>
            </div>
        {/if}

        {#if statusList.length == 0 && isLoaded === true}
            <div class="notification is-success is-light">
                <p>알려진 문제가 없습니다!</p>
            </div>
        {/if}

        {#each statusList as status}
            <div class="box">
                <h3 class="title is-3">{status.title}</h3>
                <p class="subtitle mb-1">{status.parsedDate}</p>
                <div class="tags">
                    {#each status.labels as label}
                        <span
                            class="tag label is-medium tooltip"
                            style="background-color: #{label.color}; color: #;"
                            on:click="{() => {
                                alert(label.description);
                            }}">
                            {label.name}
                            <span class="tooltip-text">
                                {label.description}
                            </span>
                        </span>
                    {/each}
                </div>
                <div class="content is-medium">{@html parse(status.body)}</div>
            </div>
        {/each}
    </div>
</section>
